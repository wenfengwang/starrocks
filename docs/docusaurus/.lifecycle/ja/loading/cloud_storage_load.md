---
displayed_sidebar: "Japanese"
---

# クラウドストレージからデータをロードします

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、次の方法のいずれかを使用してクラウドストレージから大量のデータをロードすることができます：[ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) および [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

v3.0およびそれ以前では、StarRocksはブローカーロードのみをサポートしており、非同期ロードモードで実行されます。ロードジョブを送信すると、StarRocksは非同期でジョブを実行します。`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#ロードジョブの表示)"セクションを参照してください。

ブローカーロードは、複数のデータファイルをロードする際に各ロードジョブのトランザクション的な原子性を保証し、つまり1つのロードジョブで複数のデータファイルをロードする際にはすべて成功するかすべて失敗するかのどちらかとなります。あるデータファイルのロードが成功し、他のファイルのロードが失敗することはありません。

また、ブローカーロードはデータロード時のデータ変換をサポートし、データロード中のUPSERTおよびDELETE操作によるデータ変更をサポートしています。詳細については、[ロード時のデータ変換](../loading/Etl_in_loading.md) および [ロードによるデータの変更](../loading/Load_to_Primary_Key_tables.md) を参照してください。

<InsertPrivNote />

v3.1以降、StarRocksは、外部テーブルを作成する手間を省いて、INSERTコマンドとFILESキーワードを使用してAWS S3からParquet形式またはORC形式のファイルのデータを直接ロードする機能をサポートしています。詳細については、[INSERT > ファイルを使用して外部ソースから直接データを挿入する](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-files) を参照してください。

このトピックでは、クラウドストレージからデータをロードするための [ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) の使用に焦点を当てています。

## サポートされているデータファイルフォーマット

ブローカーロードは、次のデータファイルフォーマットをサポートしています：

- CSV

- Parquet

- ORC

> **注意**
>
> CSVデータの場合、次の点に注意してください：
>
> - テキストデリミタとして長さが50バイト以下のUTF-8文字列（カンマ（,）、タブ、またはパイプ（|）など）を使用できます。
> - `\\N` を使用してヌル値を表します。たとえば、データファイルには3つの列があり、そのファイルのレコードが最初の列と3番目の列にデータを保持していますが、2番目の列にはデータがありません。この状況では、2番目の列にヌル値を表すために `\\N` を使用する必要があります。これは、レコードを `a,\\N,b` としてコンパイルする必要があることを意味します。 `a,,b` は、レコードの2番目の列に空の文字列があることを示します。

## 仕組み

ロードジョブをFEに送信すると、FEはクエリプランを生成し、利用可能なBEの数とロードしたいデータファイルのサイズに基づいてクエリプランを分割し、それぞれのクエリプランを利用可能なBEに割り当てます。ロード中、関与する各BEは、外部ストレージシステムからデータファイルのデータを取得し、データの事前処理を行い、それをStarRocksクラスタにロードします。すべてのBEがクエリプランの作業を完了した後、FEはロードジョブが成功したかどうかを判断します。

以下の図は、ブローカーロードジョブのワークフローを示しています。

![ブローカーロードのワークフロー](../assets/broker_load_how-to-work_en.png)

## データの準備例

1. ローカルのファイルシステムにログインし、2つのCSV形式のデータファイル `file1.csv` と `file2.csv` を作成します。両方のファイルは、順番にユーザーID、ユーザー名、ユーザースコアを表す3つの列で構成されています。

   - `file1.csv`

     ```Plain
     1,Lily,21
     2,Rose,22
     3,Alice,23
     4,Julia,24
     ```

   - `file2.csv`

     ```Plain
     5,Tony,25
     6,Adam,26
     7,Allen,27
     8,Jacky,28
     ```

2. `file1.csv` および `file2.csv` を、AWS S3バケット `bucket_s3` の `input` フォルダー、Google GCSバケット `bucket_gcs` の `input` フォルダー、S3互換ストレージオブジェクト（たとえばMinIO）バケット `bucket_minio` の `input` フォルダ、およびAzure Storageの指定されたパスにアップロードします。

3. 自分のStarRocksデータベース（たとえば `test_db`）にログインし、2つのプライマリキー付きテーブル `table1` および `table2` を作成します。両方のテーブルは、`id`、`name`、`score` の3つの列で構成されており、そのうち `id` がプライマリキーです。

   ```SQL
   CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "ユーザーID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "ユーザー名",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "ユーザースコア"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
             
   CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "ユーザーID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "ユーザー名",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "ユーザースコア"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
   ```

## AWS S3からデータをロードする

ブローカーロードは、S3プロトコルまたはS3Aプロトコルに従ってAWS S3へのアクセスをサポートしています。したがって、AWS S3からデータをロードする場合は、ファイルパス（DATA INFILE）に `s3://` または `s3a://` をプレフィックスとして含めることができます。

また、以下の例では、CSVファイル形式とインスタンスプロファイル認証方法を使用しています。他の形式でデータをロードする方法や、他の認証方法を使用する際に必要な認証パラメータの設定については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

### 1つのデータファイルを1つのテーブルにロードする

#### 例

次の文を実行して、AWS S3バケット `bucket_s3` の `input` フォルダーに保存された `file1.csv` のデータを `table1` にロードします。

```SQL
LOAD LABEL test_db.label_brokerloadtest_101
(
    DATA INFILE("s3a://bucket_s3/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "<aws_s3_region>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用して、ジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#ロードジョブの表示)"セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 複数のデータファイルを1つのテーブルにロードする

#### 例

次の文を実行して、AWS S3バケット `bucket_s3` の `input` フォルダーに保存されたすべてのデータファイル（`file1.csv` および `file2.csv`）のデータを `table1` にロードします：

```SQL
LOAD LABEL test_db.label_brokerloadtest_102
(
    DATA INFILE("s3a://bucket_s3/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "<aws_s3_region>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用して、ジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#ロードジョブの表示)"セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます：

```SQL
SELECT * FROM table1;
```markdown
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 行がセットに（0.01 秒）
```

### 複数のデータファイルを複数のテーブルにロードする

#### 例

次のステートメントを実行して、AWS S3 バケット `bucket_s3` 内の `input` フォルダに保存されている `file1.csv` と `file2.csv` のデータをそれぞれ `table1` と `table2` にロードします：

```SQL
LOAD LABEL test_db.label_brokerloadtest_103
(
    DATA INFILE("s3a://bucket_s3/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("s3a://bucket_s3/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "<aws_s3_region>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリできます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示](#view-a-load-job)」セクションを参照してください。

ロードジョブが正常に完了したことを確認した後、`table1` と `table2` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます：

1. `table1` をクエリ:

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 行がセットに（0.01 秒）
   ```

2. `table2` をクエリ:

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 行がセットに（0.01 秒）
   ```
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

## Microsoft Azure Storage からデータを読み込む

Microsoft Azure Storage からデータを読み込む際には、利用するアクセスプロトコルと具体的なストレージサービスに応じて使用するプレフィックスを決定する必要があります:

- Blob Storage からデータを読み込む場合、ファイルパス（`DATA INFILE`）にアクセスプロトコルに基づき `wasb://` または `wasbs://` をプレフィックスとして含める必要があります:
  - Blob Storage が HTTP を介してのアクセスのみを許可している場合は、プレフィックスとして `wasb://` を使用します。例: `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
  - Blob Storage が HTTPS を介してのアクセスのみを許可している場合は、プレフィックスとして `wasbs://` を使用します。例: `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
- Data Lake Storage Gen1 からデータを読み込む場合、ファイルパス（`DATA INFILE`）に `adl://` をプレフィックスとして含める必要があります。例: `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
- Data Lake Storage Gen2 からデータを読み込む場合、ファイルパス（`DATA INFILE`）にアクセスプロトコルに基づき `abfs://` または `abfss://` をプレフィックスとして含める必要があります:
  - Data Lake Storage Gen2 が HTTP を介してのアクセスのみを許可している場合は、プレフィックスとして `abfs://` を使用します。例: `abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。
  - Data Lake Storage Gen2 が HTTPS を介してのアクセスのみを許可している場合は、プレフィックスとして `abfss://` を使用します。例: `abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

また、以下の例では CSV ファイル形式、Azure Blob Storage、および共有キー認証方式を使用しています。他の形式でデータを読み込む方法や、他の Azure ストレージサービスや認証方式を使用する際に構成する必要がある認証パラメータについては、「[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)」を参照してください。

### 単一のデータファイルを単一のテーブルに読み込む

#### 例

次のステートメントを実行して、Azure Storage の指定されたパスに保存されている `file1.csv` のデータを `table1` に読み込みます:

```SQL
LOAD LABEL test_db.label_brokerloadtest_301
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリできます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの「[ロードジョブの表示](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認した後は、`table1` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます:

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
+------+-------+-------+
4 行が返されました (0.01 秒)
```

### 複数のデータファイルを単一のテーブルに読み込む

#### 例

次のステートメントを実行して、Azure Storage の指定されたパスに保存されているすべてのデータファイル（`file1.csv` および `file2.csv`）のデータを `table1` に読み込みます:

```SQL
LOAD LABEL test_db.label_brokerloadtest_302
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリできます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの「[ロードジョブの表示](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認した後は、`table1` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます:

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 行が返されました (0.01 秒)
```

### 複数のデータファイルを複数のテーブルに読み込む

#### 例

次のステートメントを実行して、Azure Storage の指定されたパスに保存されている `file1.csv` および `file2.csv` のデータをそれぞれ `table1` と `table2` に読み込みます:

```SQL
LOAD LABEL test_db.label_brokerloadtest_303
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリできます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの「[ロードジョブの表示](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認した後は、`table1` および `table2` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます:

1. `table1` をクエリ:

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 行が返されました (0.01 秒)
   ```

2. `table2` をクエリ:

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 行が返されました (0.01 秒)
   ```

## S3 互換ストレージシステムからデータを読み込む

以下の例では、CSV ファイル形式および MinIO ストレージシステムを使用しています。他の形式でデータを読み込む方法については、「[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)」を参照してください。

### 単一のデータファイルを単一のテーブルに読み込む

#### 例

次のステートメントを実行して、MinIO バケット `bucket_minio` の `input` フォルダに保存されている `file1.csv` のデータを `table1` に読み込みます:

```SQL
LOAD LABEL test_db.label_brokerloadtest_401
(
    DATA INFILE("obs://bucket_minio/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
```
(
    id, name, score
)
WITH BROKER
(
    "aws.s3.enable_ssl" = "false",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### クエリデータ

ロードジョブを送信した後、「`SELECT * FROM information_schema.loads`」を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

ロードジョブが正常に完了したことを確認した後、「[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)」を使用して「`table1`」のデータをクエリできます。

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 複数のデータファイルを単一のテーブルにロードする

#### 例

次の文を実行して、MinIOバケット「bucket_minio」の「input」フォルダに格納されているすべてのデータファイル（`file1.csv`と`file2.csv`）のデータを、「table1」というテーブルにロードします。

```SQL
LOAD LABEL test_db.label_brokerloadtest_402
(
    DATA INFILE("obs://bucket_minio/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "aws.s3.enable_ssl" = "false",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### クエリデータ

ロードジョブを送信した後、「`SELECT * FROM information_schema.loads`」を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

ロードジョブが正常に完了したことを確認した後、「[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)」を使用して「table1」のデータをクエリできます。

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    3 | Alice |    23 |
|    4 | Julia |    24 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 複数のデータファイルを複数のテーブルにロードする

#### 例

次の文を実行して、「bucket_minio」の「input」フォルダに格納されているすべてのデータファイル（`file1.csv`と`file2.csv`）のデータを、「table1」と「table2」というテーブルにそれぞれロードします。

```SQL
LOAD LABEL test_db.label_brokerloadtest_403
(
    DATA INFILE("obs://bucket_minio/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_minio/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    "aws.s3.enable_ssl" = "false",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### クエリデータ

ロードジョブを送信した後、「`SELECT * FROM information_schema.loads`」を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

ロードジョブが正常に完了したことを確認した後、「[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)」を使用して「table1」と「table2」のデータをクエリできます。

1. 「table1」をクエリする:

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    21 |
   |    2 | Rose  |    22 |
   |    3 | Alice |    23 |
   |    4 | Julia |    24 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

2. 「table2」をクエリする:

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | Tony  |    25 |
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

## ロードジョブの表示

`information_schema`データベースの`loads`テーブルから1つまたは複数のロードジョブの結果をクエリするために、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)文を使用します。この機能はv3.1以降でサポートされています。

例1: `test_db`データベースで実行されたロードジョブの結果をクエリする。クエリ文では、最大2つの結果を返し、結果は作成時間(`CREATE_TIME`)で降順に並べ替えられることを指定します。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db'
ORDER BY create_time DESC
LIMIT 2\G
```

以下の結果が返されます:

```SQL
*************************** 1. row ***************************
              JOB_ID: 20686
               LABEL: label_brokerload_unqualifiedtest_83
       DATABASE_NAME: test_db
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: BROKER
            PRIORITY: NORMAL
           SCAN_ROWS: 8
       FILTERED_ROWS: 0
     UNSELECTED_ROWS: 0
           SINK_ROWS: 8
            ETL_INFO:
           TASK_INFO: resource:N/A; timeout(s):14400; max_filter_ratio:1.0
         CREATE_TIME: 2023-08-02 15:25:22
      ETL_START_TIME: 2023-08-02 15:25:24
     ETL_FINISH_TIME: 2023-08-02 15:25:24
     LOAD_START_TIME: 2023-08-02 15:25:24
    LOAD_FINISH_TIME: 2023-08-02 15:25:27
         JOB_DETAILS: {"All backends":{"77fe760e-ec53-47f7-917d-be5528288c08":[10006],"0154f64e-e090-47b7-a4b2-92c2ece95f97":[10005]},"FileNumber":2,"FileSize":84,"InternalTableLoadBytes":252,"InternalTableLoadRows":8,"ScanBytes":84,"ScanRows":8,"TaskNumber":2,"Unfinished backends":{"77fe760e-ec53-47f7-917d-be5528288c08":[],"0154f64e-e090-47b7-a4b2-92c2ece95f97":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: NULL
        TRACKING_SQL: NULL
REJECTED_RECORD_PATH: NULL
*************************** 2. row ***************************
              JOB_ID: 20624
               LABEL: label_brokerload_unqualifiedtest_82
       DATABASE_NAME: test_db
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: BROKER
            PRIORITY: NORMAL
           SCAN_ROWS: 12
       FILTERED_ROWS: 4
     UNSELECTED_ROWS: 0
           SINK_ROWS: 8
            ETL_INFO:
           TASK_INFO: resource:N/A; timeout(s):14400; max_filter_ratio:1.0
         CREATE_TIME: 2023-08-02 15:23:29
```
ETL_START_TIME: 2023-08-02 15:23:34
     ETL_FINISH_TIME: 2023-08-02 15:23:34
     LOAD_START_TIME: 2023-08-02 15:23:34
    LOAD_FINISH_TIME: 2023-08-02 15:23:34
         JOB_DETAILS: {"All backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[10010],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[10006]},"FileNumber":2,"FileSize":158,"InternalTableLoadBytes":333,"InternalTableLoadRows":8,"ScanBytes":158,"ScanRows":12,"TaskNumber":2,"Unfinished backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: http://172.26.195.69:8540/api/_load_error_log?file=error_log_78f78fc38509451f_a0a2c6b5db27dcb7
        TRACKING_SQL: select tracking_log from information_schema.load_tracking_logs where job_id=20624
REJECTED_RECORD_PATH: 172.26.95.92:/home/disk1/sr/be/storage/rejected_record/test_db/label_brokerload_unqualifiedtest_0728/6/404a20b1e4db4d27_8aa9af1e8d6d8bdc