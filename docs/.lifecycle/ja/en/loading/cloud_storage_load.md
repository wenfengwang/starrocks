---
displayed_sidebar: English
---

# クラウドストレージからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、クラウドストレージから大量のデータをロードするために、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)と[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)のいずれかの方法を使用することをサポートしています。

v3.0以前では、StarRocksはBroker Loadのみをサポートしており、これは非同期ロードモードで実行されます。ロードジョブを送信した後、StarRocksはジョブを非同期的に実行します。`SELECT * FROM information_schema.loads`を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

Broker Loadは、複数のデータファイルをロードする各ジョブのトランザクションの原子性を保証します。つまり、1つのロードジョブ内の複数のデータファイルのロードは全て成功するか、または失敗する必要があります。一部のデータファイルのロードが成功し、他のファイルのロードが失敗することはありません。

さらに、Broker Loadはデータロード時のデータ変換をサポートし、データロード中のUPSERTおよびDELETE操作によるデータ変更もサポートします。詳細については、[ロード時のデータ変換](../loading/Etl_in_loading.md)と[ロードを通じたデータ変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

<InsertPrivNote />

v3.1以降、StarRocksはINSERTコマンドとFILESキーワードを使用して、AWS S3からParquet形式またはORC形式のファイルデータを直接ロードすることをサポートしています。これにより、最初に外部テーブルを作成する手間が省けます。詳細については、[INSERT > FILESキーワードを使用して外部ソースのファイルから直接データを挿入する](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-files)を参照してください。

このトピックでは、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用してクラウドストレージからデータをロードする方法に焦点を当てています。

## サポートされているデータファイル形式

Broker Loadは、以下のデータファイル形式をサポートしています：

- CSV

- Parquet

- ORC

> **注記**
>
> CSVデータについては、以下の点に注意してください：
>
> - コンマ（,）、タブ、パイプ（|）など、50バイトを超えないUTF-8文字列をテキスト区切り文字として使用できます。
> - Null値は`\N`を使用して表されます。例えば、データファイルが3つの列で構成されており、そのデータファイルのレコードが1列目と3列目にデータを持ち、2列目にデータがない場合、2列目に`\N`を使用してnull値を示す必要があります。つまり、レコードは`a,\N,b`としてコンパイルされるべきであり、`a,,b`ではありません。`a,,b`はレコードの2列目が空文字列を持つことを意味します。

## 仕組み

ロードジョブをFEに送信すると、FEはクエリプランを生成し、利用可能なBEの数とロードするデータファイルのサイズに基づいてクエリプランを分割し、各部分を利用可能なBEに割り当てます。ロード中、各BEは外部ストレージシステムからデータファイルのデータを取得し、データを前処理した後、StarRocksクラスタにロードします。すべてのBEがクエリプランの部分を完了すると、FEはロードジョブが成功したかどうかを判断します。

以下の図は、Broker Loadジョブのワークフローを示しています。

![Broker Loadのワークフロー](../assets/broker_load_how-to-work_en.png)

## データ例の準備

1. ローカルファイルシステムにログインし、`file1.csv`と`file2.csv`という2つのCSV形式のデータファイルを作成します。どちらのファイルも、ユーザーID、ユーザー名、ユーザースコアを順に表す3つの列で構成されています。

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

2. `file1.csv`と`file2.csv`をAWS S3バケット`bucket_s3`の`input`フォルダ、Google GCSバケット`bucket_gcs`の`input`フォルダ、S3互換ストレージオブジェクト（例えばMinIO）バケット`bucket_minio`の`input`フォルダ、およびAzure Storageの指定されたパスにアップロードします。

3. StarRocksデータベース（例えば`test_db`）にログインし、`table1`と`table2`という2つのプライマリキーテーブルを作成します。どちらのテーブルも、`id`、`name`、`score`の3つの列で構成され、`id`がプライマリキーです。

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

Broker Loadは、S3またはS3Aプロトコルに従ってAWS S3にアクセスすることをサポートしているため、AWS S3からデータをロードする際には、ファイルパス（`DATA INFILE`）として渡すS3 URIに`s3://`または`s3a://`のプレフィックスを含めることができます。

また、以下の例ではCSVファイル形式とインスタンスプロファイルベースの認証方法を使用しています。他の形式でデータをロードする方法や、他の認証方法を使用する際に設定する必要がある認証パラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

次のステートメントを実行して、AWS S3バケット`bucket_s3`の`input`フォルダに保存されている`file1.csv`のデータを`table1`にロードします。

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

#### データクエリ

ロードジョブを送信した後、`SELECT * FROM information_schema.loads`を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して`table1`のデータをクエリできます。

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

次のステートメントを実行して、AWS S3バケット`bucket_s3`の`input`フォルダに保存されているすべてのデータファイル（`file1.csv`と`file2.csv`）のデータを`table1`にロードします。

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

#### データクエリ

ロードジョブを送信した後、`SELECT * FROM information_schema.loads`を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して`table1`のデータをクエリできます。

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

次のステートメントを実行して、AWS S3バケット `bucket_s3` の `input` フォルダに保存されている `file1.csv` と `file2.csv` のデータをそれぞれ `table1` と `table2` にロードします。

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

ロードジョブをサブミットした後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` と `table2` のデータをクエリできます。

1. `table1` をクエリする:

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

2. `table2` をクエリする:

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

## Google GCSからデータをロードする

Broker Loadはgsプロトコルに従ってのみGoogle GCSへのアクセスをサポートしているため、Google GCSからデータをロードする際は、ファイルパス(`DATA INFILE`)として渡すGCS URIに`gs://`の接頭辞を含める必要があります。

また、以下の例ではCSVファイル形式とVMベースの認証方法を使用しています。他の形式でデータをロードする方法や、他の認証方法を使用する際に設定する必要がある認証パラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

次のステートメントを実行して、Google GCSバケット `bucket_gcs` の `input` フォルダに保存されている `file1.csv` のデータを `table1` にロードします。

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

ロードジョブをサブミットした後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` のデータをクエリできます。

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

次のステートメントを実行して、Google GCSバケット `bucket_gcs` の `input` フォルダに保存されているすべてのデータファイル(`file1.csv` と `file2.csv`)のデータを `table1` にロードします。

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

ロードジョブをサブミットした後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` のデータをクエリできます。

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

次のステートメントを実行して、Google GCSバケット `bucket_gcs` の `input` フォルダに保存されている `file1.csv` と `file2.csv` のデータをそれぞれ `table1` と `table2` にロードします。

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

ロードジョブをサブミットした後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` と `table2` のデータをクエリできます。

1. `table1` をクエリする:

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

2. `table2` をクエリする:

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

## Microsoft Azure Storageからデータをロードする

Azure Storageからデータをロードする際は、使用するアクセスプロトコルと特定のストレージサービスに基づいて使用するプレフィックスを決定する必要があります。

- Blob Storageからデータをロードする場合、ストレージアカウントへのアクセスに使用されるプロトコルに基づいて、ファイルパス(`DATA INFILE`)に`wasb://` または `wasbs://` のプレフィックスを含める必要があります。
  - Blob StorageがHTTPを介してのみアクセスを許可している場合は、`wasb://` をプレフィックスとして使用します。例: `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
  - Blob StorageがHTTPSを介してのみアクセスを許可している場合は、`wasbs://` をプレフィックスとして使用します。例: `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
- Data Lake Storage Gen1からデータをロードする場合、ファイルパス(`DATA INFILE`)に`adl://` のプレフィックスを含める必要があります。例: `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
- Data Lake Storage Gen2からデータをロードする場合、ストレージアカウントへのアクセスに使用されるプロトコルに基づいて、ファイルパス(`DATA INFILE`)に`abfs://` または `abfss://` のプレフィックスを含める必要があります。
  - Data Lake Storage Gen2がHTTPを介してのみアクセスを許可している場合は、`abfs://` をプレフィックスとして使用します。例: `abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

  - Data Lake Storage Gen2 が HTTPS 経由でのみアクセスを許可している場合は、プレフィックスとして `abfss://` を使用します（例: `abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`）。

また、以下の例ではCSVファイル形式、Azure Blob Storage、および共有キーに基づく認証方法を使用しています。他の形式でデータをロードする方法や、他のAzureストレージサービスおよび認証方法を使用する際に設定が必要な認証パラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

以下のステートメントを実行して、Azure Storageの指定されたパスに保存されている `file1.csv` のデータを `table1` にロードします：

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

#### データクエリ

ロードジョブを提出した後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` のデータをクエリできます：

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

以下のステートメントを実行して、Azure Storageの指定されたパスに保存されているすべてのデータファイル（`file1.csv` と `file2.csv`）のデータを `table1` にロードします：

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

#### データクエリ

ロードジョブを提出した後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` のデータをクエリできます：

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

### 複数のデータファイルを複数のテーブルにロードする

#### 例

以下のステートメントを実行して、Azure Storageの指定されたパスに保存されている `file1.csv` のデータを `table1` に、`file2.csv` のデータを `table2` にそれぞれロードします：

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

#### データクエリ

ロードジョブを提出した後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` と `table2` のデータをクエリできます：

1. `table1` のクエリ：

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

2. `table2` のクエリ：

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

## S3互換ストレージシステムからのデータロード

以下の例ではCSVファイル形式とMinIOストレージシステムを使用しています。他の形式でデータをロードする方法については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### 単一のデータファイルを単一のテーブルにロードする

#### 例

以下のステートメントを実行して、MinIOバケット `bucket_minio` の `input` フォルダに保存されている `file1.csv` のデータを `table1` にロードします：

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

#### データクエリ

ロードジョブを提出した後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` のデータをクエリできます：

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

以下のステートメントを実行して、MinIOバケット `bucket_minio` の `input` フォルダに保存されているすべてのデータファイル（`file1.csv` と `file2.csv`）のデータを `table1` にロードします：

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

#### データクエリ

ロードジョブを提出した後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能はv3.1以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` のデータをクエリできます：

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

### 複数のデータファイルを複数のテーブルにロードする

#### 例

次のステートメントを実行して、MinIO バケット `bucket_minio` の `input` フォルダに保存されているすべてのデータファイル（`file1.csv` と `file2.csv`）のデータをそれぞれ `table1` と `table2` にロードします:

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

ロードジョブをサブミットした後、`SELECT * FROM information_schema.loads` を使用してジョブ結果を照会できます。この機能は v3.1 以降でサポートされています。詳細については、このトピックの「[ロードジョブを表示する](#view-a-load-job)」セクションを参照してください。

ロードジョブが成功したことを確認したら、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) を使用して `table1` と `table2` のデータをクエリできます:

1. `table1` をクエリする:

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

2. `table2` をクエリする:

   ```SQL
   SELECT * FROM table2;
   +------+-------+-------+
   | id   | city  |
   +------+-------+-------+
   |    5 | Tokyo |
   |    6 | Osaka |
   |    7 | Kyoto |
   |    8 | Nagoya|
   +------+-------+-------+
   4 rows in set (0.01 sec)
   ```

## ロードジョブを表示する

[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して、`information_schema` データベースの `loads` テーブルから1つ以上のロードジョブの結果を照会します。この機能は v3.1 以降でサポートされています。

例 1: `test_db` データベースで実行されたロードジョブの結果を照会します。クエリステートメントで、最大2つの結果を返すことができ、返される結果は作成時間 (`CREATE_TIME`) で降順に並べ替えられる必要があることを指定します。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db'
ORDER BY create_time DESC
LIMIT 2\G
```

次の結果が返されます:

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

例 2: `test_db` データベースで実行されたロードジョブ（ラベルが `label_brokerload_unqualifiedtest_82`）の結果を照会します:

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'test_db' and label = 'label_brokerload_unqualifiedtest_82'\G
```

次の結果が返されます:

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

戻り値のフィールドの詳細については、[情報スキーマ > loads](../reference/information_schema/loads.md) を参照してください。

## ロードジョブをキャンセルする

ロードジョブが **CANCELLED** または **FINISHED** ステージにない場合、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) ステートメントを使用してジョブをキャンセルできます。

たとえば、次のステートメントを実行して、データベース `test_db` のラベル `label1` のロードジョブをキャンセルできます:

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label1";
```

## ジョブの分割と並行実行

Broker Load ジョブは、同時に実行される1つ以上のタスクに分割できます。ロードジョブ内のタスクは、単一のトランザクション内で実行されます。それらはすべて成功するか失敗するかのどちらかです。StarRocksは、`LOAD` ステートメント内の `data_desc` の宣言に基づいて各ロードジョブを分割します:

- 複数の `data_desc` パラメーターを宣言し、それぞれが異なるテーブルを指定する場合、各テーブルのデータをロードするタスクが生成されます。

- 複数の `data_desc` パラメーターを宣言し、それぞれが同じテーブルの異なるパーティションを指定する場合、各パーティションのデータをロードするタスクが生成されます。

さらに、各タスクは1つ以上のインスタンスにさらに分割され、StarRocksクラスターのBEに均等に分散して同時に実行されます。StarRocksは、以下の [FE設定](../administration/FE_configuration.md#fe-configuration-items) に基づいて各タスクを分割します:

- `min_bytes_per_broker_scanner`: 各インスタンスが処理するデータの最小量。デフォルトは64MBです。

- `load_parallel_instance_num`: 個々のBEで各ロードジョブに許可される並行インスタンスの数。デフォルトは1です。

  個々のタスクでのインスタンス数を計算するには、以下の式を使用できます。

  **個々のタスクのインスタンス数 = min(個々のタスクが読み込むデータ量/`min_bytes_per_broker_scanner`, `load_parallel_instance_num` x BEの数)**

ほとんどのケースでは、各ロードジョブに対して`data_desc`は1つだけ宣言され、各ロードジョブは1つのタスクにのみ分割され、そのタスクはBEの数と同じ数のインスタンスに分割されます。

## トラブルシューティング

[Broker Load FAQ](../faq/loading/Broker_load_faq.md)を参照してください。
