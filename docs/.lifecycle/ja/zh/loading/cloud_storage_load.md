---
displayed_sidebar: Chinese
---

# クラウドストレージからのインポート

StarRocksは、クラウドストレージシステムから大量のデータをインポートするための2つの方法をサポートしています：[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) と [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

バージョン3.0以前では、StarRocksはBroker Loadによるインポート方法のみをサポートしていました。Broker Loadは非同期インポート方法で、インポートジョブを提出した後、StarRocksは非同期でインポートジョブを実行します。`SELECT * FROM information_schema.loads` を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされており、詳細は本文の「[インポートジョブの確認](#インポートジョブの確認)」セクションを参照してください。

Broker Loadは、一度のインポートトランザクションの原子性を保証できます。つまり、一度のインポートで複数のデータファイルがすべて成功するか、すべて失敗するかのどちらかであり、一部が成功し一部が失敗するという状況は発生しません。

また、Broker Loadはインポートプロセス中にデータ変換を行うこと、およびUPSERTとDELETE操作を通じてデータ変更を実現することもサポートしています。詳細は[インポートプロセス中のデータ変換](../loading/Etl_in_loading.md)と[インポートを通じたデータ変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

> **注意**
>
> Broker Load操作には対象テーブルのINSERT権限が必要です。もしINSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。

バージョン3.1から、StarRocksはINSERT文と`FILES`キーワードを使用して、AWS S3から直接ParquetまたはORC形式のデータファイルをインポートすることをサポートし、外部テーブルを事前に作成する手間を省きました。詳細は[INSERT > FILESキーワードを使用して外部データファイルを直接インポートする](../loading/InsertInto.md#FILESキーワードを使用して外部データファイルを直接インポートする)を参照してください。

この記事では、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用してクラウドストレージシステムからデータをインポートする方法について主に説明します。

## サポートされるデータファイル形式

Broker Loadは以下のデータファイル形式をサポートしています：

- CSV

- Parquet

- ORC

> **説明**
>
> CSV形式のデータについては、以下の2点に注意が必要です：
>
> - StarRocksは、最大50バイトのUTF-8エンコーディング文字列を列の区切り文字として設定することをサポートしており、一般的なカンマ(,)、タブ、パイプ(|)が含まれます。
> - 空値(null)は`\N`で表されます。例えば、データファイルに3列があり、ある行の第1列と第3列のデータがそれぞれ`a`と`b`で、第2列にデータがない場合、第2列は空値を`\N`で表し、`a,\N,b`と記述します。`a,,b`は第2列が空の文字列であることを意味します。

## 基本原理

インポートジョブを提出した後、FEは対応するクエリプランを生成し、現在利用可能なBEの数とソースデータファイルのサイズに基づいて、複数のBEにクエリプランを割り当てて実行します。各BEはインポートタスクの一部を担当します。BEは実行中に外部ストレージシステムからデータを取得し、データの前処理を行った後にStarRocksにデータをインポートします。すべてのBEがインポートを完了した後、FEが最終的にインポートジョブが成功したかどうかを判断します。

以下の図はBroker Loadの主要なプロセスを示しています：

![Broker Loadの原理図](../assets/broker_load_how-to-work_zh.png)

## データサンプルの準備

1. ローカルファイルシステムに、2つのCSV形式のデータファイル`file1.csv`と`file2.csv`を作成します。2つのデータファイルはそれぞれ3列を含み、ユーザーID、ユーザー名、ユーザースコアを表しています。以下のようになります：

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

2. `file1.csv`と`file2.csv`をクラウドストレージスペースの指定されたパスにアップロードします。ここでは、AWS S3の`bucket_s3`、Google GCSの`bucket_gcs`、アリババクラウドOSSの`bucket_oss`、テンセントクラウドCOSの`bucket_cos`、ファーウェイクラウドOBSの`bucket_obs`、その他のS3プロトコル互換オブジェクトストレージ（例えばMinIO）の`bucket_minio`、およびAzure Storageの指定されたパスの`input`フォルダにそれぞれアップロードされたと仮定します。

3. StarRocksデータベース（`test_db`と仮定）にログインし、`table1`と`table2`の2つのプライマリキーモデルテーブルを作成します。2つのテーブルはそれぞれ`id`、`name`、`score`の3列を含み、ユーザーID、ユーザー名、ユーザースコアを表しており、プライマリキーは`id`列です。以下のようになります：

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

## AWS S3からのインポート

Broker LoadはS3またはS3Aプロトコルを介してAWS S3にアクセスすることをサポートしているため、AWS S3からデータをインポートする際には、ファイルパス（`DATA INFILE`）に指定されたターゲットファイルのS3 URIに`s3://`または`s3a://`をプレフィックスとして使用できます。

また、以下のコマンドはCSV形式とInstance Profileに基づく認証方法を例にしています。他の形式のデータをインポートする方法や、他の認証方法を使用する際に必要なパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、AWS S3の`bucket_s3`にある`input`フォルダ内のデータファイル`file1.csv`のデータをターゲットテーブル`table1`にインポートします：

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

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされており、詳細は本文の「[インポートジョブの確認](#インポートジョブの確認)」セクションを参照してください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`のデータをクエリすることができます。以下のようになります：

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

### 複数のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、AWS S3の`bucket_s3`にある`input`フォルダ内のすべてのデータファイル（`file1.csv`と`file2.csv`）のデータをターゲットテーブル`table1`にインポートします：

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

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされており、詳細は本文の「[インポートジョブの確認](#インポートジョブの確認)」セクションを参照してください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`のデータをクエリすることができます。以下のようになります：

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
8 rows in set (0.01 sec)
```

### 複数のデータファイルを複数のテーブルにインポート

#### 操作例

以下のステートメントを使用して、AWS S3の`bucket_s3`にある`input`フォルダ内のデータファイル`file1.csv`と`file2.csv`のデータをそれぞれターゲットテーブル`table1`と`table2`にインポートします：

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

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされており、詳細は本文の「[インポートジョブの確認](#インポートジョブの確認)」セクションを参照してください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`と`table2`のデータをクエリすることができます：

1. `table1`のデータをクエリするには、以下のようにします：

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
   4行がセットされました (0.01秒)
   ```

2. `table2`のデータを以下のようにクエリします:

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
   4行がセットされました (0.01秒)
   ```

## Google GCSからのインポート

注意: Broker Loadは`gs`プロトコルを通じてのみGoogle GCSにアクセスをサポートしているため、Google GCSからデータをインポートする際には、ファイルパス(`DATA INFILE`)に指定されたターゲットファイルのGCS URIが`gs://`をプレフィックスとして使用していることを確認する必要があります。

また、以下のコマンドはCSV形式とVMベースの認証方式を例としています。他の形式のデータをインポートする方法や、他の認証方式を使用する際に設定が必要なパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、Google GCSの`bucket_gcs`内の`input`フォルダにあるデータファイル`file1.csv`のデータをターゲットテーブル`table1`にインポートします:

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

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[インポートジョブの確認](#インポートジョブの確認)」セクションを参照してください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`のデータをクエリすることができます。以下のように表示されます:

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
4行がセットされました (0.01秒)
```

### 複数のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、Google GCSの`bucket_gcs`内の`input`フォルダにあるすべてのデータファイル(`file1.csv`と`file2.csv`)のデータをターゲットテーブル`table1`にインポートします:

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

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[インポートジョブの確認](#インポートジョブの確認)」セクションを参照してください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`のデータをクエリすることができます。以下のように表示されます:

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
4行がセットされました (0.01秒)
```

### 複数のデータファイルを複数のテーブルにインポート

#### 操作例

以下のステートメントを使用して、Google GCSの`bucket_gcs`内の`input`フォルダにあるデータファイル`file1.csv`と`file2.csv`のデータをそれぞれターゲットテーブル`table1`と`table2`にインポートします:

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
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[インポートジョブの確認](#インポートジョブの確認)」セクションを参照してください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`と`table2`のデータをクエリすることができます:

1. `table1`のデータを以下のようにクエリします:

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
   4行がセットされました (0.01秒)
   ```

2. `table2`のデータを以下のようにクエリします:

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
   4行がセットされました (0.01秒)
   ```

## Microsoft Azure Storageからのインポート

注意: Azure Storageからデータをインポートする際には、使用するアクセスプロトコルとストレージサービスに応じてファイルパス(`DATA INFILE`)のプレフィックスを決定する必要があります:

- Blob Storageからデータをインポートする場合、ファイルパス(`DATA INFILE`)に`wasb://`または`wasbs://`をプレフィックスとして追加する必要があります:
  - HTTPプロトコルを使用してアクセスする場合は、`wasb://`をプレフィックスとして使用します。例: `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
  - HTTPSプロトコルを使用してアクセスする場合は、`wasbs://`をプレフィックスとして使用します。例: `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
- Azure Data Lake Storage Gen1からデータをインポートする場合、ファイルパス(`DATA INFILE`)に`adl://`をプレフィックスとして追加する必要があります。例: `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
- Data Lake Storage Gen2からデータをインポートする場合、ファイルパス(`DATA INFILE`)に`abfs://`または`abfss://`をプレフィックスとして追加する必要があります:
  - HTTPプロトコルを使用してアクセスする場合は、`abfs://`をプレフィックスとして使用します。例: `abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。
  - HTTPSプロトコルを使用してアクセスする場合は、`abfss://`をプレフィックスとして使用します。例: `abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

また、以下のコマンドはCSV形式、Azure Blob Storage、Shared Keyベースの認証方式を例としています。他の形式のデータをインポートする方法や、他のAzureオブジェクトストレージサービスを使用する際に設定が必要なパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、Azure Storageの指定されたパスにあるデータファイル`file1.csv`のデータをターゲットテーブル`table1`にインポートします:

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

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[インポートジョブの確認](#インポートジョブの確認)」セクションを参照してください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`のデータをクエリすることができます。以下のように表示されます:

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
4行がセットされました (0.01秒)
```

### 複数のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、Azure Storageの指定されたパスにあるすべてのデータファイル(`file1.csv`と`file2.csv`)のデータをターゲットテーブル`table1`にインポートします:

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

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[インポートジョブの確認](#インポートジョブの確認)」セクションを参照してください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`のデータをクエリすることができます。以下のように表示されます:

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
4行がセットされました (0.01秒)
```

### 複数のデータファイルを複数のテーブルにインポートする

#### 操作例

以下のステートメントを使用して、Azure Storageの指定されたパスにあるデータファイル`file1.csv`と`file2.csv`をそれぞれ`table1`と`table2`にインポートします：

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

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は「[インポートジョブの表示](#インポートジョブの表示)」セクションをご覧ください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`と`table2`のデータをクエリできます：

1. `table1`のデータをクエリするには、以下のようにします：

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
   4行がセットされました (0.01秒)
   ```

2. `table2`のデータをクエリするには、以下のようにします：

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
   4行がセットされました (0.01秒)
   ```

## 阿里云OSSからインポート

以下のコマンドはCSV形式の例です。他の形式のデータをインポートする方法については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにインポートする

#### 操作例

以下のステートメントを使用して、阿里云OSSの`bucket_oss`内の`input`フォルダにあるデータファイル`file1.csv`を`table1`にインポートします：

```SQL
LOAD LABEL test_db.label_brokerloadtest_401
(
    DATA INFILE("oss://bucket_oss/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.oss.accessKeyId" = "<oss_access_key>",
    "fs.oss.accessKeySecret" = "<oss_secret_key>",
    "fs.oss.endpoint" = "<oss_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は「[インポートジョブの表示](#インポートジョブの表示)」セクションをご覧ください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`のデータをクエリできます：

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
4行がセットされました (0.01秒)
```

### 複数のデータファイルを単一のテーブルにインポートする

#### 操作例

以下のステートメントを使用して、阿里云OSSの`bucket_oss`内の`input`フォルダにあるすべてのデータファイル（`file1.csv`および`file2.csv`）を`table1`にインポートします：

```SQL
LOAD LABEL test_db.label_brokerloadtest_402
(
    DATA INFILE("oss://bucket_oss/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.oss.accessKeyId" = "<oss_access_key>",
    "fs.oss.accessKeySecret" = "<oss_secret_key>",
    "fs.oss.endpoint" = "<oss_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は「[インポートジョブの表示](#インポートジョブの表示)」セクションをご覧ください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`のデータをクエリできます：

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
4行がセットされました (0.01秒)
```

### 複数のデータファイルを複数のテーブルにインポートする

#### 操作例

以下のステートメントを使用して、阿里云OSSの`bucket_oss`内の`input`フォルダにあるデータファイル`file1.csv`と`file2.csv`をそれぞれ`table1`と`table2`にインポートします：

```SQL
LOAD LABEL test_db.label_brokerloadtest_403
(
    DATA INFILE("oss://bucket_oss/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("oss://bucket_oss/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.oss.accessKeyId" = "<oss_access_key>",
    "fs.oss.accessKeySecret" = "<oss_secret_key>",
    "fs.oss.endpoint" = "<oss_endpoint>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は「[インポートジョブの表示](#インポートジョブの表示)」セクションをご覧ください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`と`table2`のデータをクエリできます：

1. `table1`のデータをクエリするには、以下のようにします：

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
   4行がセットされました (0.01秒)
   ```

2. `table2`のデータをクエリするには、以下のようにします：

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
   4行がセットされました (0.01秒)
   ```

## 腾讯云COSからインポート

以下のコマンドはCSV形式の例です。他の形式のデータをインポートする方法については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにインポートする

#### 操作例

以下のステートメントを使用して、腾讯云COSの`bucket_cos`内の`input`フォルダにあるデータファイル`file1.csv`を`table1`にインポートします：

```SQL
LOAD LABEL test_db.label_brokerloadtest_501
(
    DATA INFILE("cosn://bucket_cos/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.cosn.userinfo.secretId" = "<cos_access_key>",
    "fs.cosn.userinfo.secretKey" = "<cos_secret_key>",
    "fs.cosn.bucket.endpoint_suffix" = "<cos_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は「[インポートジョブの表示](#インポートジョブの表示)」セクションをご覧ください。

インポートジョブが成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して`table1`のデータをクエリできます：

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
4行がセットされました (0.01秒)
```

### 複数のデータファイルを単一のテーブルにインポートする

#### 操作例

以下のステートメントを使用して、腾讯云COSの`bucket_cos`内の`input`フォルダにあるすべてのデータファイル（`file1.csv`および`file2.csv`）を`table1`にインポートします：

```SQL
LOAD LABEL test_db.label_brokerloadtest_502
(
    DATA INFILE("cosn://bucket_cos/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.cosn.userinfo.secretId" = "<cos_access_key>",
    "fs.cosn.userinfo.secretKey" = "<cos_secret_key>",
    "fs.cosn.bucket.endpoint_suffix" = "<cos_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

インポートジョブを提出した後、`SELECT * FROM information_schema.loads`を使用してBroker Loadジョブの結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は「[インポートジョブの表示](#インポートジョブの表示)」セクションをご覧ください。


作業を提出した後、`SELECT * FROM information_schema.loads` を使用して Broker Load の結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[作業の確認](#作業の確認)」をご覧ください。

作業が成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して `table1` のデータを以下のようにクエリできます:

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

### 複数のデータファイルを複数のテーブルにインポート

#### 操作例

以下のステートメントを使用して、腾讯云COSのストレージスペース `bucket_cos` の `input` フォルダにあるデータファイル `file1.csv` と `file2.csv` をそれぞれ `table1` と `table2` にインポートします:

```SQL
LOAD LABEL test_db.label_brokerloadtest_503
(
    DATA INFILE("cosn://bucket_cos/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("cosn://bucket_cos/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.cosn.userinfo.secretId" = "<cos_access_key>",
    "fs.cosn.userinfo.secretKey" = "<cos_secret_key>",
    "fs.cosn.bucket.endpoint_suffix" = "<cos_endpoint>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

作業を提出した後、`SELECT * FROM information_schema.loads` を使用して Broker Load の結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[作業の確認](#作業の確認)」をご覧ください。

作業が成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して `table1` と `table2` のデータをクエリできます:

1. `table1` のデータを以下のようにクエリします:

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

2. `table2` のデータを以下のようにクエリします:

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

## 华为云OBSからのインポート

以下のコマンドはCSV形式を例にしています。他の形式のデータをインポートする方法については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

### 単一のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、华为云OBSのストレージスペース `bucket_obs` の `input` フォルダにあるデータファイル `file1.csv` を `table1` にインポートします:

```SQL
LOAD LABEL test_db.label_brokerloadtest_601
(
    DATA INFILE("obs://bucket_obs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.obs.access.key" = "<obs_access_key>",
    "fs.obs.secret.key" = "<obs_secret_key>",
    "fs.obs.endpoint" = "<obs_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

作業を提出した後、`SELECT * FROM information_schema.loads` を使用して Broker Load の結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[作業の確認](#作業の確認)」をご覧ください。

作業が成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して `table1` のデータを以下のようにクエリできます:

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

### 複数のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、华为云OBSのストレージスペース `bucket_obs` の `input` フォルダにあるすべてのデータファイル（`file1.csv` と `file2.csv`）を `table1` にインポートします:

```SQL
LOAD LABEL test_db.label_brokerloadtest_602
(
    DATA INFILE("obs://bucket_obs/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.obs.access.key" = "<obs_access_key>",
    "fs.obs.secret.key" = "<obs_secret_key>",
    "fs.obs.endpoint" = "<obs_endpoint>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

作業を提出した後、`SELECT * FROM information_schema.loads` を使用して Broker Load の結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[作業の確認](#作業の確認)」をご覧ください。

作業が成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して `table1` のデータを以下のようにクエリできます:

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

### 複数のデータファイルを複数のテーブルにインポート

#### 操作例

以下のステートメントを使用して、华为云OBSのストレージスペース `bucket_obs` の `input` フォルダにあるデータファイル `file1.csv` と `file2.csv` をそれぞれ `table1` と `table2` にインポートします:

```SQL
LOAD LABEL test_db.label_brokerloadtest_603
(
    DATA INFILE("obs://bucket_obs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_obs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER
(
    "fs.obs.access.key" = "<obs_access_key>",
    "fs.obs.secret.key" = "<obs_secret_key>",
    "fs.obs.endpoint" = "<obs_endpoint>"
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

作業を提出した後、`SELECT * FROM information_schema.loads` を使用して Broker Load の結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[作業の確認](#作業の確認)」をご覧ください。

作業が成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して `table1` と `table2` のデータをクエリできます:

1. `table1` のデータを以下のようにクエリします:

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

2. `table2` のデータを以下のようにクエリします:

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

## S3プロトコル互換の他のオブジェクトストレージからのインポート

以下のコマンドはCSV形式とMinIOストレージを例にしています。他の形式のデータをインポートする方法については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。

### 単一のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、MinIOのストレージスペース `bucket_minio` の `input` フォルダにあるデータファイル `file1.csv` を `table1` にインポートします:

```SQL
LOAD LABEL test_db.label_brokerloadtest_701
(
    DATA INFILE("s3://bucket_minio/input/file1.csv")
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

作業を提出した後、`SELECT * FROM information_schema.loads` を使用して Broker Load の結果を確認できます。この機能はバージョン3.1からサポートされています。詳細は本文の「[作業の確認](#作業の確認)」をご覧ください。

作業が成功したことを確認した後、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して `table1` のデータを以下のようにクエリできます:

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

### 複数のデータファイルを単一のテーブルにインポート

#### 操作例

以下のステートメントを使用して、MinIOのストレージスペース `bucket_minio` の `input` フォルダにあるすべてのデータファイル（`file1.csv` と `file2.csv`）を `table1` にインポートします:

```SQL
LOAD LABEL test_db.label_brokerloadtest_702
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

インポートジョブを提出した後、`SELECT * FROM information_schema.loads` を使用して Broker Load ジョブの結果を確認できます。この機能はバージョン 3.1 からサポートされており、詳細は「[インポートジョブの表示](#インポートジョブの表示)」セクションを参照してください。

インポートジョブが成功したことを確認した後、以下のように [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して `table1` のデータをクエリできます：

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

### 複数のデータファイルを複数のテーブルにインポート

#### 操作例

以下のステートメントを使用して、MinIO ストレージスペース `bucket_minio` の `input` フォルダ内のデータファイル `file1.csv` と `file2.csv` のデータをそれぞれターゲットテーブル `table1` と `table2` にインポートします：

```SQL
LOAD LABEL test_db.label_brokerloadtest_703
(
    DATA INFILE("obs://bucket_minio/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_minio/input/file2.csv")
    INTO TABLE table2
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
);
PROPERTIES
(
    "timeout" = "3600"
);
```

#### データのクエリ

インポートジョブを提出した後、`SELECT * FROM information_schema.loads` を使用して Broker Load ジョブの結果を確認できます。この機能はバージョン 3.1 からサポートされており、詳細は「[インポートジョブの表示](#インポートジョブの表示)」セクションを参照してください。

インポートジョブが成功したことを確認した後、以下のように [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して `table1` と `table2` のデータをクエリできます：

1. `table1` のデータをクエリするには、以下のようにします：

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

2. `table2` のデータをクエリするには、以下のようにします：

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

## インポートジョブの表示

バージョン 3.1 からサポートされている `information_schema` データベースの `loads` テーブルから [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して Broker Load ジョブの結果を確認できます。

例1：以下のコマンドを使用して `test_db` データベースのインポートジョブの実行状況を確認し、作業の作成時間 (`CREATE_TIME`) に基づいて結果を降順で並べ替え、最大2つの結果データを表示します：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db'
ORDER BY create_time DESC
LIMIT 2\G
```

結果は以下のように表示されます：

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

例2：以下のコマンドを使用して `test_db` データベースの `label_brokerload_unqualifiedtest_82` ラベルのインポートジョブの実行状況を確認します：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db' and label = 'label_brokerload_unqualifiedtest_82'\G
```

結果は以下のように表示されます：

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

`information_schema.loads` に関する詳細なフィールドの説明は、[`information_schema.loads`](../reference/information_schema/loads.md) を参照してください。

## インポートジョブのキャンセル

インポートジョブの状態が **CANCELLED** または **FINISHED** でない場合、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) ステートメントを使用してそのインポートジョブをキャンセルできます。

例えば、以下のステートメントを使用して `test_db` データベースの `label1` ラベルのインポートジョブをキャンセルできます：

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label1";
```

## ジョブの分割と並行実行

Broker Load ジョブは、一つまたは複数のサブタスクに分割されて並行して処理されます。一つのジョブの全てのサブタスクは、トランザクション全体として成功または失敗します。ジョブの分割は `LOAD LABEL` ステートメントの `data_desc` パラメータを通じて指定されます：

- 複数の `data_desc` パラメータが宣言され、複数の異なるテーブルにデータをインポートする場合、各テーブルのデータインポートは一つのサブタスクに分割されます。

- 複数の `data_desc` パラメータを宣言して同一テーブルの異なるパーティションに対応する場合、各パーティションのデータインポートは個別のサブタスクに分割されます。

各サブタスクはさらに一つまたは複数のインスタンスに分割され、これらのインスタンスは均等に BE 上で並行実行されます。インスタンスの分割は以下の [FE 設定](../administration/FE_configuration.md#設定-fe-動的パラメータ)によって決定されます:

- `min_bytes_per_broker_scanner`：単一インスタンスが処理する最小データ量で、デフォルトは 64 MB です。

- `load_parallel_instance_num`：単一の BE で許可される各ジョブの並行インスタンス数で、デフォルトは 1 です。バージョン 3.1 以降では非推奨となります。

   以下の式を使用して、単一サブタスクのインスタンス総数を計算できます：

   単一サブタスクのインスタンス総数 = min（サブタスクのインポート対象データ総量 / `min_bytes_per_broker_scanner`、`load_parallel_instance_num` x BE の総数）

通常、インポートジョブには一つの `data_desc` のみがあり、一つのサブタスクに分割され、サブタスクは BE の総数に等しいインスタンスに分割されます。

## よくある質問

[Broker Load よくある質問](../faq/loading/Broker_load_faq.md)をご覧ください。
