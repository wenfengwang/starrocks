---
displayed_sidebar: "Japanese"
---

# クラウドストレージからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、クラウドストレージから大量のデータをロードするために、次の方法のいずれかを使用することができます：[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) および [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

v3.0 およびそれ以前では、StarRocksは Broker Load のみをサポートしており、非同期ロードモードで実行されます。ロードジョブを送信すると、StarRocksは非同期でジョブを実行します。ジョブの結果をクエリするには、`SELECT * FROM information_schema.loads` を使用できます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

Broker Load は、複数のデータファイルをロードするために実行される各ロードジョブのトランザクション的な原子性を保証します。つまり、1つのロードジョブで複数のデータファイルをロードする場合、すべてのデータファイルのロードが成功するか、すべて失敗するかのいずれかです。一部のデータファイルのロードが成功し、他のファイルのロードが失敗することはありません。

さらに、Broker Load はデータロード時のデータ変換をサポートし、データロード中に UPSERT および DELETE 操作によるデータの変更をサポートします。詳細については、[ロード時のデータ変換](../loading/Etl_in_loading.md) および [ロードを介したデータの変更](../loading/Load_to_Primary_Key_tables.md) を参照してください。

<InsertPrivNote />

v3.1 以降、StarRocksは INSERT コマンドと FILES キーワードを使用して、Parquet 形式または ORC 形式のファイルのデータを AWS S3 から直接ロードすることをサポートしています。これにより、最初に外部テーブルを作成する手間が省けます。詳細については、[INSERT > FILES キーワードを使用して外部ソースのファイルからデータを直接挿入する](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-files) を参照してください。

このトピックでは、クラウドストレージからデータをロードするための [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) の使用方法に焦点を当てています。

## サポートされるデータファイル形式

Broker Load は、次のデータファイル形式をサポートしています：

- CSV

- Parquet

- ORC

> **注意**
>
> CSV データの場合、次の点に注意してください：
>
> - カンマ (,)、タブ、またはパイプ (|) など、長さが 50 バイトを超えない UTF-8 文字列をテキスト区切り文字として使用できます。
> - Null 値は `\N` を使用して示します。たとえば、データファイルは 3 つの列から構成され、そのデータファイルのレコードは最初の列と 3 番目の列にデータを保持していますが、2 番目の列にはデータがありません。この場合、2 番目の列には `\N` を使用して null 値を示す必要があります。つまり、レコードは `a,\N,b` ではなく `a,,b` とコンパイルする必要があります。`a,,b` は、レコードの 2 番目の列に空の文字列が含まれていることを示します。

## 動作原理

FE にロードジョブを送信すると、FE はクエリプランを生成し、利用可能な BE の数とロードするデータファイルのサイズに基づいてクエリプランを分割し、各クエリプランの一部を利用可能な BE に割り当てます。ロード中、関連するすべての BE は外部ストレージシステムからデータファイルのデータを取得し、データを前処理して StarRocks クラスタにデータをロードします。すべての BE がクエリプランの各部分を完了した後、FE はロードジョブが成功したかどうかを判断します。

次の図は、Broker Load ジョブのワークフローを示しています。

![Broker Load のワークフロー](../assets/broker_load_how-to-work_en.png)

## データの準備例

1. ローカルファイルシステムにログインし、2 つの CSV 形式のデータファイル `file1.csv` と `file2.csv` を作成します。両方のファイルは、ユーザー ID、ユーザー名、およびユーザースコアを順に表す 3 つの列で構成されます。

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

2. `file1.csv` と `file2.csv` を AWS S3 バケット `bucket_s3` の `input` フォルダ、Google GCS バケット `bucket_gcs` の `input` フォルダ、S3 互換ストレージオブジェクト（MinIO など）バケット `bucket_minio` の `input` フォルダ、および Azure Storage の指定されたパスにアップロードします。

3. StarRocks データベース（たとえば `test_db`）にログインし、2 つのプライマリキーテーブル `table1` と `table2` を作成します。両方のテーブルは、`id`、`name`、および `score` の 3 つの列で構成されます。`id` はプライマリキーです。

   ```SQL
   CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "ユーザー ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "ユーザー名",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "ユーザースコア"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
             
   CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "ユーザー ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "ユーザー名",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "ユーザースコア"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
   ```

## AWS S3 からデータをロードする

Broker Load は、S3 または S3A プロトコルに基づいて AWS S3 へのアクセスをサポートしています。したがって、AWS S3 からデータをロードする場合は、ファイルパス (`DATA INFILE`) として S3 URI に `s3://` または `s3a://` を含めることができます。

また、以下の例では、CSV ファイル形式とインスタンスプロファイルベースの認証方法を使用しています。他の形式でデータをロードする方法や、他の認証方法を使用する場合に設定する必要のある認証パラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

`file1.csv` のデータを AWS S3 バケット `bucket_s3` の `input` フォルダに格納されているデータを `table1` にロードするには、次のステートメントを実行します：

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

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

### 複数のデータファイルを単一のテーブルにロードする

#### 例

`file1.csv` および `file2.csv` のデータを AWS S3 バケット `bucket_s3` の `input` フォルダに格納されているデータを `table1` にロードするには、次のステートメントを実行します：

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

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
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 複数のデータファイルを複数のテーブルにロードする

#### 例

`file1.csv` および `file2.csv` のデータを AWS S3 バケット `bucket_s3` の `input` フォルダに格納されているデータを `table1` および `table2` にそれぞれロードするには、次のステートメントを実行します：

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1` および `table2` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます：

1. `table1` をクエリする：

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

2. `table2` をクエリする：

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

## Google GCS からデータをロードする

Google GCS からデータをロードする場合、gs プロトコルのみを使用できます。したがって、Google GCS からデータをロードする場合は、ファイルパス (`DATA INFILE`) に `gs://` をプレフィックスとして含める必要があります。

また、以下の例では、CSV ファイル形式と VM ベースの認証方法を使用しています。他の形式でデータをロードする方法や、他の認証方法を使用する場合に設定する必要のある認証パラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

`file1.csv` のデータを Google GCS バケット `bucket_gcs` の `input` フォルダに格納されているデータを `table1` にロードするには、次のステートメントを実行します：

```SQL
LOAD LABEL test_db.label_brokerloadtest_201
(
    DATA INFILE("gs://bucket_gcs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "gcp.gcs.use_compute_engine_service_account" = "true"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

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

### 複数のデータファイルを単一のテーブルにロードする

#### 例

`file1.csv` および `file2.csv` のデータを Google GCS バケット `bucket_gcs` の `input` フォルダに格納されているデータを `table1` にロードするには、次のステートメントを実行します：

```SQL
LOAD LABEL test_db.label_brokerloadtest_202
(
    DATA INFILE("gs://bucket_gcs/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "gcp.gcs.use_compute_engine_service_account" = "true"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

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
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 複数のデータファイルを複数のテーブルにロードする

#### 例

`file1.csv` および `file2.csv` のデータを Google GCS バケット `bucket_gcs` の `input` フォルダに格納されているデータを `table1` および `table2` にそれぞれロードするには、次のステートメントを実行します：

```SQL
LOAD LABEL test_db.label_brokerloadtest_203
(
    DATA INFILE("gs://bucket_gcs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("gs://bucket_gcs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "gcp.gcs.use_compute_engine_service_account" = "true"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1` および `table2` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます：

1. `table1` をクエリする：

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

2. `table2` をクエリする：

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

## Microsoft Azure Storage からデータをロードする

Azure Storage からデータをロードする場合、使用するアクセスプロトコルと具体的なストレージサービスに基づいて、どのプレフィックスを使用するかを決定する必要があります：

- Blob Storage からデータをロードする場合、ストレージアカウントへのアクセスに使用するプロトコルに基づいて、ファイルパス (`DATA INFILE`) に `wasb://` または `wasbs://` をプレフィックスとして含める必要があります：
  - Blob Storage が HTTP を介してのアクセスのみを許可している場合は、`wasb://` をプレフィックスとして使用します。たとえば、`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*` のようになります。
  - Blob Storage が HTTPS を介してのアクセスのみを許可している場合は、`wasbs://` をプレフィックスとして使用します。たとえば、`wasbs://<container>@<storage_account>.blob.core.windows``.net/<path>/<file_name>/*` のようになります。
- Data Lake Storage Gen1 からデータをロードする場合、ファイルパス (`DATA INFILE`) に `adl://` をプレフィックスとして含める必要があります。たとえば、`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>` のようになります。
- Data Lake Storage Gen2 からデータをロードする場合、使用するアクセスプロトコルに基づいて、ファイルパス (`DATA INFILE`) に `abfs://` または `abfss://` をプレフィックスとして含める必要があります：
  - Data Lake Storage Gen2 が HTTP を介してのアクセスのみを許可している場合は、`abfs://` をプレフィックスとして使用します。たとえば、`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>` のようになります。
  - Data Lake Storage Gen2 が HTTPS を介してのアクセスのみを許可している場合は、`abfss://` をプレフィックスとして使用します。たとえば、`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>` のようになります。

また、以下の例では、CSV ファイル形式、Azure Blob Storage、および共有キーベースの認証方法を使用しています。他の形式でデータをロードする方法や、他の Azure ストレージサービスおよび認証方法を使用する場合に設定する必要のある認証パラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

指定されたパスの Azure Storage に格納されている `file1.csv` のデータを `table1` にロードするには、次のステートメントを実行します：

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

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

### 複数のデータファイルを単一のテーブルにロードする

#### 例

指定されたパスの Azure Storage に格納されているすべてのデータファイル (`file1.csv` および `file2.csv`) のデータを `table1` にロードするには、次のステートメントを実行します：

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

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
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
4 rows in set (0.01 sec)
```

### 複数のデータファイルを複数のテーブルにロードする

#### 例

指定されたパスの Azure Storage に格納されている `file1.csv` および `file2.csv` のデータを `table1` および `table2` にそれぞれロードするには、次のステートメントを実行します：

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1` および `table2` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます：

1. `table1` をクエリする：

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

2. `table2` をクエリする：

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

## S3 互換ストレージシステムからデータをロードする

以下の例では、CSV ファイル形式と MinIO ストレージシステムを使用しています。他の形式でデータをロードする方法については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

`file1.csv` のデータを MinIO バケット `bucket_minio` の `input` フォルダに格納されているデータを `table1` にロードするには、次のステートメントを実行します：

```SQL
LOAD LABEL test_db.label_brokerloadtest_401
(
    DATA INFILE("obs://bucket_minio/input/file1.csv")
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

#### データのクエリ

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリすることができます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの "[ロードジョブの表示](#ロードジョブの表示)" セクションを参照してください。

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

### 複数のデータファイルを単一のテーブルにロードする

#### 例

次のステートメントを実行して、MinIOバケット「bucket_minio」の「input」フォルダに保存されているすべてのデータファイル（`file1.csv`および`file2.csv`）のデータを「table1」にロードします。

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

#### データのクエリ

ロードジョブを送信した後、「SELECT * FROM information_schema.loads」を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブの表示](#ロードジョブの表示)」セクションを参照してください。

ロードジョブが正常に完了したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して「table1」のデータをクエリできます。

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

次のステートメントを実行して、MinIOバケット「bucket_minio」の「input」フォルダに保存されているすべてのデータファイル（`file1.csv`および`file2.csv`）のデータをそれぞれ「table1」と「table2」にロードします。

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

#### データのクエリ

ロードジョブを送信した後、「SELECT * FROM information_schema.loads」を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブの表示](#ロードジョブの表示)」セクションを参照してください。

ロードジョブが正常に完了したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して「table1」と「table2」のデータをクエリできます。

1. 「table1」をクエリする場合：

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

2. 「table2」をクエリする場合：

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

[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して、`information_schema`データベースの`loads`テーブルから1つ以上のロードジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。

例1：`test_db`データベースで実行されたロードジョブの結果をクエリする場合。クエリステートメントでは、最大2つの結果が返され、返される結果は作成時刻（`CREATE_TIME`）の降順でソートされるように指定します。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db'
ORDER BY create_time DESC
LIMIT 2\G
```

次の結果が返されます。

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
      ETL_START_TIME: 2023-08-02 15:23:34
     ETL_FINISH_TIME: 2023-08-02 15:23:34
     LOAD_START_TIME: 2023-08-02 15:23:34
    LOAD_FINISH_TIME: 2023-08-02 15:23:34
         JOB_DETAILS: {"All backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[10010],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[10006]},"FileNumber":2,"FileSize":158,"InternalTableLoadBytes":333,"InternalTableLoadRows":8,"ScanBytes":158,"ScanRows":12,"TaskNumber":2,"Unfinished backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: http://172.26.195.69:8540/api/_load_error_log?file=error_log_78f78fc38509451f_a0a2c6b5db27dcb7
        TRACKING_SQL: select tracking_log from information_schema.load_tracking_logs where job_id=20624
REJECTED_RECORD_PATH: 172.26.95.92:/home/disk1/sr/be/storage/rejected_record/test_db/label_brokerload_unqualifiedtest_0728/6/404a20b1e4db4d27_8aa9af1e8d6d8bdc
```

例2：`test_db`データベースで実行されたロードジョブ（ラベルが「label_brokerload_unqualifiedtest_82」）の結果をクエリする場合：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db' and label = 'label_brokerload_unqualifiedtest_82'\G
```

次の結果が返されます。

```SQL
*************************** 1. row ***************************
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
      ETL_START_TIME: 2023-08-02 15:23:34
     ETL_FINISH_TIME: 2023-08-02 15:23:34
     LOAD_START_TIME: 2023-08-02 15:23:34
    LOAD_FINISH_TIME: 2023-08-02 15:23:34
         JOB_DETAILS: {"All backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[10010],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[10006]},"FileNumber":2,"FileSize":158,"InternalTableLoadBytes":333,"InternalTableLoadRows":8,"ScanBytes":158,"ScanRows":12,"TaskNumber":2,"Unfinished backends":{"78f78fc3-8509-451f-a0a2-c6b5db27dcb6":[],"a24aa357-f7de-4e49-9e09-e98463b5b53c":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: http://172.26.195.69:8540/api/_load_error_log?file=error_log_78f78fc38509451f_a0a2c6b5db27dcb7
        TRACKING_SQL: select tracking_log from information_schema.load_tracking_logs where job_id=20624
REJECTED_RECORD_PATH: 172.26.95.92:/home/disk1/sr/be/storage/rejected_record/test_db/label_brokerload_unqualifiedtest_0728/6/404a20b1e4db4d27_8aa9af1e8d6d8bdc
```

返される結果のフィールドについての詳細は、[Information Schema > loads](../reference/information_schema/loads.md)を参照してください。

## ロードジョブのキャンセル

ロードジョブが「CANCELLED」または「FINISHED」のステージにない場合、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)ステートメントを使用してジョブをキャンセルできます。

たとえば、データベース「test_db」でラベルが「label1」のロードジョブをキャンセルするには、次のステートメントを実行します。

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label1";
```

## ジョブの分割と並行実行

Broker Loadジョブは、1つ以上のタスクに分割されて並行実行されることがあります。タスク内のタスクは、単一のトランザクション内で実行されます。すべてが成功するか、すべてが失敗する必要があります。StarRocksは、`LOAD`ステートメントで`data_desc`を宣言する方法に基づいて、各ロードジョブを分割します。

- 異なるテーブルを指定する個々の`data_desc`パラメータを宣言する場合、各テーブルのデータをロードするためのタスクが生成されます。

- 同じテーブルの異なるパーティションを指定する個々の`data_desc`パラメータを宣言する場合、各パーティションのデータをロードするためのタスクが生成されます。

さらに、各タスクは1つ以上のインスタンスにさらに分割され、これらのインスタンスはStarRocksクラスタのBEに均等に分散して並行実行されます。StarRocksは、各タスクを次の[FEの設定](../administration/Configuration.md#fe-configuration-items)に基づいて分割します。

- `min_bytes_per_broker_scanner`：各インスタンスで処理される最小のデータ量。デフォルトの量は64 MBです。

- `load_parallel_instance_num`：各ロードジョブで個々のBEで許可される並行インスタンスの数。デフォルトの数は1です。

  個々のタスク内のインスタンスの数を計算するためには、次の式を使用できます。

  **個々のタスク内のインスタンスの数 = 個々のタスクでロードするデータ量/`min_bytes_per_broker_scanner`、`load_parallel_instance_num` x BEの数**

ほとんどの場合、各ロードジョブには1つの`data_desc`のみが宣言され、各ロードジョブは1つのタスクにのみ分割され、タスクはBEの数と同じ数のインスタンスに分割されます。

## トラブルシューティング

「[Broker Load FAQ](../faq/loading/Broker_load_faq.md)」を参照してください。
