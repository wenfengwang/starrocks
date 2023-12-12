---
displayed_sidebar: "Japanese"
---

# クラウドストレージからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、クラウドストレージから大量のデータをロードするために次のいずれかの方法を使用することをサポートしています: [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) および [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

v3.0以前では、StarRocksはBroker Loadのみをサポートし、非同期ロードモードで実行されます。ロードジョブを送信すると、StarRocksは非同期でジョブを実行します。`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされます。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

Broker Loadは、複数のデータファイルをロードするために実行される各ロードジョブのトランザクション的な原子性を保証します。これは、1つのロードジョブで複数のデータファイルをロードする際に、すべてのデータファイルのロードが成功するか失敗するかであることを意味します。いくつかのデータファイルのロードが成功しても、他のファイルのロードが失敗することはありません。

さらに、Broker Loadはデータロード時のデータ変換をサポートし、UPSERTやDELETE操作によるデータ変更もサポートします。詳細については、[ロード時のデータ変換](../loading/Etl_in_loading.md)および[ロードによるデータ変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

<InsertPrivNote />

v3.1以降、StarRocksは、外部テーブルを作成する手間を省くために、INSERTコマンドとFILESキーワードを使用してAWS S3からParquet形式またはORC形式のファイルのデータを直接ロードすることをサポートしています。詳細については、[INSERT > FILESキーワードを使用して外部ソースのファイルからデータを直接挿入](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-files)を参照してください。

このトピックでは、クラウドストレージからデータをロードするために[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用することに焦点を当てています。

## サポートされているデータファイルフォーマット

Broker Loadは、次のデータファイルフォーマットをサポートしています:

- CSV

- Parquet

- ORC

> **注記**
>
> CSVデータの場合、以下の点に注意してください:
>
> - テキスト区切り記号として、コンマ(,)、タブ、またはパイプ(|)などの長さが50バイトを超えないUTF-8文字列を使用できます。
> - ヌル値は`\N`を使用して示します。たとえば、データファイルは3つの列からなり、そのデータファイルのレコードは最初と3番目の列にデータを保持していますが、2番目の列にはデータがありません。この場合、2番目の列にヌル値を示すために`\N`を使用する必要があります。これは、レコードは`a,\N,b`としてコンパイルされなければならず、`a,,b`ではなく`a,,b`となります。`a,,b`は、レコードの2番目の列が空の文字列を保持していることを示します。

## 動作仕様

ロードジョブをFEに送信すると、FEはクエリプランを生成し、利用可能なBEの数とロードしたいデータファイルのサイズに基づいてクエリプランをポーションに分割し、それぞれのクエリプランを利用可能なBEに割り当てます。ロード中、関与する各BEは、外部ストレージシステムからデータファイルのデータを取得し、データを前処理し、その後データをStarRocksクラスタにロードします。すべてのBEがクエリプランのそれぞれのポーションを完了した後、FEはロードジョブが成功したかどうかを決定します。

以下の図はBroker Loadジョブのワークフローを示しています。

![Broker Loadのワークフロー](../assets/broker_load_how-to-work_en.png)

## データの例を準備する

1. ローカルのファイルシステムにログインし、`file1.csv`および`file2.csv`という2つのCSV形式のデータファイルを作成します。両方のファイルはユーザーID、ユーザー名、および順番にユーザースコアを表す3つの列で構成されています。

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

2. `file1.csv`および`file2.csv`をAWS S3バケット`bucket_s3`の`input`フォルダに、Google GCSバケット`bucket_gcs`の`input`フォルダに、S3互換ストレージオブジェクト(MinIOなど)のバケット`bucket_minio`の`input`フォルダに、およびAzure Storageの指定されたパスにアップロードします。

3. StarRocksデータベース(例: `test_db`)にログインし、`table1`および`table2`という2つのプライマリキーテーブルを作成します。両方のテーブルは、`id`、`name`、および`score`の3つの列で構成されており、`id`がプライマリキーです。

   ```SQL
   CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "user name",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "user score"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
             
   CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "user name",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "user score"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
   ```

## AWS S3からデータをロードする

Broker Loadは、S3またはS3Aプロトコルに準拠したAWS S3へのアクセスをサポートしています。したがって、AWS S3からデータをロードする場合は、ファイルパス(`DATA INFILE`)として`S3://`または`S3a://`を含めることができます。

また、以下の例はCSVファイル形式とインスタンスプロファイルベースの認証方法を使用しています。他のファイル形式でデータをロードする方法や他の認証方法を使用する際に構成する必要がある認証パラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

以下のステートメントを実行して、AWS S3バケット`bucket_s3`の`input`フォルダに保存されている`file1.csv`のデータを`table1`にロードします:

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

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされます。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1`のデータをクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます:

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

以下のステートメントを実行して、AWS S3バケット`bucket_s3`の`input`フォルダに保存されているすべてのデータファイル(`file1.csv`および`file2.csv`)のデータを`table1`にロードします:

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

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされます。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1`のデータをクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます:

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | リリー |    21 |
|    2 | ローズ |    22 |
|    3 | アリス |    23 |
|    4 | ジュリア |    24 |
|    5 | トニー |    25 |
|    6 | アダム |    26 |
|    7 | アレン |    27 |
|    8 | ジャッキー |    28 |
+------+-------+-------+
4 行が選択されました (0.01 秒)

### 複数のデータファイルを複数のテーブルに読み込む

#### 例

次のステートメントを実行して、AWS S3バケット`bucket_s3`の`input`フォルダに保存されている`file1.csv`および`file2.csv`のデータを、それぞれ`table1`と`table2`に読み込みます。

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

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

ロードジョブが成功したことを確認した後は、`table1`および`table2`のデータをクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます。

1. `table1`をクエリ:

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | リリー |    21 |
   |    2 | ローズ |    22 |
   |    3 | アリス |    23 |
   |    4 | ジュリア |    24 |
   +------+-------+-------+
   4 行が選択されました (0.01 秒)
   ```

2. `table2`をクエリ:

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | トニー |    25 |
   |    6 | アダム |    26 |
   |    7 | アレン |    27 |
   |    8 | ジャッキー |    28 |
   +------+-------+-------+
   4 行が選択されました (0.01 秒)
   ```

## Google GCSからデータをロード

ブローカーロードは、gsプロトコルに従ってGoogle GCSへのアクセスのみをサポートしています。そのため、Google GCSからデータをロードする際は、ファイルパス（`DATA INFILE`）としてGCS URIに`gs://`を含める必要があります。

また、以下の例では、CSVファイル形式およびVMベースの認証メソッドが使用されています。他の形式でデータをロードしたり、他の認証方法を使用する際に構成する必要がある認証パラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにロード

#### 例

次のステートメントを実行して、Google GCSバケット`bucket_gcs`の`input`フォルダに保存されている`file1.csv`のデータを`table1`にロードします。

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

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

ロードジョブが成功したことを確認した後は、`table1`のデータをクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます。

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | リリー |    21 |
|    2 | ローズ |    22 |
|    3 | アリス |    23 |
|    4 | ジュリア |    24 |
+------+-------+-------+
4 行が選択されました (0.01 秒)
```

### 複数のデータファイルを単一のテーブルにロード

#### 例

次のステートメントを実行して、Google GCSバケット`bucket_gcs`の`input`フォルダに保存されているすべてのデータファイル（`file1.csv`および`file2.csv`）のデータを`table1`にロードします。

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

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

ロードジョブが成功したことを確認した後は、`table1`のデータをクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます。

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | リリー |    21 |
|    2 | ローズ |    22 |
|    3 | アリス |    23 |
|    4 | ジュリア |    24 |
|    5 | トニー |    25 |
|    6 | アダム |    26 |
|    7 | アレン |    27 |
|    8 | ジャッキー |    28 |
+------+-------+-------+
4 行が選択されました (0.01 秒)
```

### 複数のデータファイルを複数のテーブルにロード

#### 例

次のステートメントを実行して、Google GCSバケット`bucket_gcs`の`input`フォルダに保存されている`file1.csv`および`file2.csv`のデータを、それぞれ`table1`と`table2`にロードします。

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

ロードジョブを送信した後は、`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブの表示](#view-a-load-job)"セクションを参照してください。

ロードジョブが成功したことを確認した後は、`table1`および`table2`のデータをクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます。

1. `table1`をクエリ:

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | リリー |    21 |
   |    2 | ローズ |    22 |
   |    3 | アリス |    23 |
   |    4 | ジュリア |    24 |
   +------+-------+-------+
   4 行が選択されました (0.01 秒)
   ```

2. `table2`をクエリ:

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    5 | トニー |    25 |
   |    6 | アダム |    26 |
   |    7 | アレン |    27 |
   |    8 | ジャッキー |    28 |
   +------+-------+-------+
   4 行が選択されました (0.01 秒)
   ```
   |    6 | Adam  |    26 |
   |    7 | Allen |    27 |
   |    8 | Jacky |    28 |
   +------+-------+-------+
   4行がセットされました（0.01秒）

## Microsoft Azure Storageからデータをロードする

Microsoft Azure Storageからデータをロードする際には、アクセスプロトコルと使用する特定のストレージサービスに基づいてどのプレフィックスを使用するかを決定する必要があります。

- Blob Storageからデータをロードする場合、ファイルパス（`DATA INFILE`）にアクセスするために使用されるプロトコルに基づいて、`wasb://`または`wasbs://`をプレフィックスとして含める必要があります。
  - Blob StorageがHTTP経由でのアクセスのみを許可する場合は、プレフィックスとして`wasb://`を使用します。たとえば`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
  - Blob StorageがHTTPS経由でのアクセスのみを許可する場合は、プレフィックスとして`wasbs://`を使用します。たとえば`wasbs://<container>@<storage_account>.blob.core.windows``.net/<path>/<file_name>/*`
- Data Lake Storage Gen1からデータをロードする場合、ファイルパス（`DATA INFILE`）に`adl://`をプレフィックスとして含める必要があります。たとえば`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
- Data Lake Storage Gen2からデータをロードする場合、ファイルパス（`DATA INFILE`）にアクセスするために使用されるプロトコルに基づいて`abfs://`または`abfss://`をプレフィックスとして含める必要があります。
  - Data Lake Storage Gen2がHTTP経由のアクセスのみを許可する場合は、プレフィックスとして`abfs://`を使用します。たとえば`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。
  - Data Lake Storage Gen2がHTTPS経由のアクセスのみを許可する場合は、プレフィックスとして`abfss://`を使用します。たとえば`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

また、次の例では、CSVファイル形式、Azure Blob Storage、共有キーベースの認証方法を使用しています。他のフォーマットでデータをロードする方法や、他のAzureストレージサービスと認証方法を使用する際に構成する必要がある認証パラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

次のステートメントを実行して、Azure Storageの指定されたパスに保存されている`file1.csv`のデータを`table1`にロードします。

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブを表示する](#ロードジョブを表示する)"セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1`のデータをクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます。

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
4行がセットされました（0.01秒）
```

### 複数のデータファイルを単一のテーブルにロードする

#### 例

次のステートメントを実行して、Azure Storageの指定されたパスに保存されているすべてのデータファイル（`file1.csv`および`file2.csv`）のデータを`table1`にロードします。

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブを表示する](#ロードジョブを表示する)"セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1`のデータをクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます。

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
8行がセットされました（0.01秒）
```

### 複数のデータファイルを複数のテーブルにロードする

#### 例

次のステートメントを実行して、Azure Storageの指定されたパスに保存されている`file1.csv`および`file2.csv`のデータをそれぞれ`table1`および`table2`にロードします。

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads`を使用してジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。詳細については、このトピックの"[ロードジョブを表示する](#ロードジョブを表示する)"セクションを参照してください。

ロードジョブが成功したことを確認した後、`table1`および`table2`のデータをクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用できます。

1. `table1`をクエリする：

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
   4行がセットされました（0.01秒）
   ```

2. `table2`をクエリする：

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
   4行がセットされました（0.01秒）
   ```

## S3互換ストレージシステムからデータをロードする

次の例では、CSVファイル形式とMinIOストレージシステムを使用しています。他のフォーマットでデータをロードする方法については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

次のステートメントを実行して、MinIOのバケット`bucket_minio`の`input`フォルダに保存されている`file1.csv`のデータを`table1`にロードします。

```SQL
LOAD LABEL test_db.label_brokerloadtest_401
(
    DATA INFILE("obs://bucket_minio/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
```
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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリできます。この機能は v3.1 以降でサポートされています。詳細は、このトピックの"[ロードジョブを表示](#ロードジョブを表示)"セクションを参照してください。

ロードジョブが成功したことを確認したら、`table1` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます。

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

### 複数のデータファイルを単一のテーブルにロードする

#### 例

以下のステートメントを実行して、MinIO バケット `bucket_minio` の `input` フォルダに格納されているすべてのデータファイル (`file1.csv` と `file2.csv`) のデータを `table1` にロードします：

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリできます。この機能は v3.1 以降でサポートされています。詳細は、このトピックの"[ロードジョブを表示](#ロードジョブを表示)"セクションを参照してください。

ロードジョブが成功したことを確認したら、`table1` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます。

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

### 複数のデータファイルを複数のテーブルにロードする

#### 例

以下のステートメントを実行して、MinIO バケット `bucket_minio` の `input` フォルダに格納されているすべてのデータファイル (`file1.csv` と `file2.csv`) のデータを、それぞれ `table1` と `table2` にロードします：

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

ロードジョブを送信した後、`SELECT * FROM information_schema.loads` を使用してジョブの結果をクエリできます。この機能は v3.1 以降でサポートされています。詳細は、このトピックの"[ロードジョブを表示](#ロードジョブを表示)"セクションを参照してください。

ロードジョブが成功したことを確認したら、`table1` と `table2` のデータをクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用できます。

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
   4 行が返されました (0.01 秒)
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
   4 行が返されました (0.01 秒)
   ```

## ロードジョブを表示

`information_schema` データベースの `loads` テーブルから、1 つ以上のロードジョブの結果をクエリするために [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用します。この機能は v3.1 以降でサポートされています。

例 1: `test_db` データベースで実行されたロードジョブの結果をクエリする。クエリステートメントでは、最大 2 つの結果が返され、結果は作成時刻（`CREATE_TIME`）で降順にソートされるように指定します。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db'
ORDER BY create_time DESC
LIMIT 2\G
```

次の結果が返されます：

```SQL
*************************** 1 行目 ***************************
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
         JOB_DETAILS: {"All backends":{"77fe760e-ec53-47f7-917d-be5528288c08":[10006],"0154f64e-e090-47b7-a4b2-92c2ece95f97":[10005]},"FileNumber":2,"FileSize":84,"InternalTableLoadBytes":252,"InternalTableLoadRows":8,"ScanBytes":84,"ScanRows: 8,"TaskNumber":2," Unfinished backends:{"77fe760e-ec53-47f7-917d-be5528288c08":[],"0154f64e-e090-47b7-a4b2-92c2ece95f97":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: NULL
        TRACKING_SQL: NULL
REJECTED_RECORD_PATH: NULL
*************************** 2 行目 ***************************
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