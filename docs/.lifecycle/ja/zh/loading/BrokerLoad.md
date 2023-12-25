---
displayed_sidebar: Chinese
---

# HDFSまたは外部クラウドストレージシステムからのデータインポート

StarRocksは、MySQLプロトコルに基づいたBroker Loadインポート方式を提供し、HDFSまたは外部クラウドストレージシステムから大量のデータをインポートするのに役立ちます。

Broker Loadは非同期のインポート方式です。インポートジョブを送信した後、StarRocksは非同期でインポートジョブを実行します。インポートジョブの結果を確認するには、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) ステートメントまたはcurlコマンドを使用します。

Broker Loadは、シングルテーブルロード（Single-Table Load）とマルチテーブルロード（Multi-Table Load）をサポートしています。一度のインポート操作で、1つまたは複数のデータファイルを1つまたは複数のターゲットテーブルにインポートできます。Broker Loadは、1回のインポートトランザクションの原子性を保証します。つまり、複数のデータファイルがすべて成功するか、すべて失敗するかのどちらかであり、一部が成功し一部が失敗するということはありません。

Broker Loadは、インポートプロセス中にデータ変換を行うことをサポートし、UPSERTおよびDELETE操作を通じてデータ変更を実現します。詳細は[データ変換の実装](./Etl_in_loading.md)と[インポートによるデータ変更の実装](../loading/Load_to_Primary_Key_tables.md)を参照してください。

> **注意**
>
> Broker Load操作には、ターゲットテーブルのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を参照してユーザーに権限を付与してください。

## 背景情報

v2.4以前のバージョンでは、StarRocksがBroker Loadを実行する際にはBrokerの助けを借りて外部ストレージシステムにアクセスする必要があり、「Brokerを使用したインポート」と呼ばれていました。インポートステートメントでは、`WITH BROKER "<broker_name>"` を使用してどのBrokerを使用するかを指定する必要がありました。Brokerは独立したステートレスサービスで、ファイルシステムインターフェースをカプセル化しています。Brokerを介して、StarRocksは外部ストレージシステム上のデータファイルにアクセスして読み取り、データファイル内のデータを事前処理してインポートすることができます。

v2.5以降、StarRocksはBroker Loadを実行する際にBrokerを使用せずに外部ストレージシステムにアクセスできるようになり、「Brokerを使用しないインポート」と呼ばれています。インポートステートメントでは`broker_name`を指定する必要はなくなりましたが、`WITH BROKER`キーワードは引き続き保持されています。

ただし、HDFSをデータソースとする一部のシナリオでは、Brokerを使用しないインポートが制限される場合があります。例えば、複数のHDFSクラスターや複数のKerberosユーザーが存在するシナリオなどです。これらのシナリオでは、Brokerを使用したインポートを続けることができ、少なくとも1組の独立したBrokerがデプロイされていることを確認する必要があります。認証方法とHA設定を指定する方法については、[HDFS](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md##hdfs)を参照してください。

> **説明**
>
> [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) ステートメントを使用して、StarRocksクラスターにデプロイされているBrokerを確認できます。クラスターにBrokerがデプロイされていない場合は、[Brokerノードのデプロイ](../deployment/deploy_broker.md)を参照してBrokerのデプロイを完了してください。

## サポートされているデータファイル形式

Broker Loadは以下のデータファイル形式をサポートしています：

- CSV

- Parquet

- ORC

> **説明**
>
> CSV形式のデータについては、以下の2点に注意する必要があります：
>
> - StarRocksは、最大50バイトのUTF-8エンコード文字列を列の区切り文字として設定することをサポートしており、カンマ(,)、タブ、パイプ(|)などの一般的な区切り文字を含みます。
> - 空値(null)は`\N`で表されます。例えば、データファイルに3列があり、ある行のデータの第1列と第3列がそれぞれ`a`と`b`で、第2列にデータがない場合、第2列は空値として`\N`で表され、`a,\N,b`と記述されます。`a,,b`は第2列が空文字列であることを意味します。

## サポートされている外部ストレージシステム

Broker Loadは、以下の外部ストレージシステムからデータをインポートすることをサポートしています：

- HDFS

- AWS S3

- Google GCS

- アリババクラウドOSS

- テンセントクラウドCOS

- ファーウェイクラウドOBS

- S3プロトコルと互換性のあるその他のオブジェクトストレージ（例：MinIO）

- Microsoft Azure Storage

## 基本原理

インポートジョブを送信した後、FEは対応するクエリプランを生成し、利用可能なBEの数とソースデータファイルのサイズに基づいて、複数のBEにクエリプランを割り当てて実行します。各BEはインポートタスクの一部を担当します。BEは実行中にHDFSまたはクラウドストレージシステムからデータを取得し、データを事前処理した後にStarRocksにインポートします。すべてのBEがインポートを完了した後、FEが最終的にインポートジョブが成功したかどうかを判断します。

以下の図は、Broker Loadの主要なプロセスを示しています：

![Broker Loadの原理図](../assets/broker_load_how-to-work_zh.png)

## 基本操作

### マルチテーブルロード（Multi-Table Load）ジョブの作成

ここではCSV形式のデータを例に、複数のデータファイルを複数のターゲットテーブルにインポートする方法について説明します。他の形式のデータをインポートする方法や、Broker Loadの詳細な構文とパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

StarRocksでは、一部の単語がSQL言語の予約キーワードであり、SQLステートメントで直接使用することはできません。これらの予約キーワードをSQLステートメントで使用する場合は、バッククォート(`)で囲む必要があります。[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データ例

1. ローカルファイルシステムにCSV形式のデータファイルを作成します。

   a. `file1.csv`という名前のデータファイルを作成します。ファイルには3列が含まれ、それぞれユーザーID、ユーザー名、ユーザースコアを表しています。以下のようになります：

      ```Plain
      1,Lily,23
      2,Rose,23
      3,Alice,24
      4,Julia,25
      ```

   b. `file2.csv`という名前のデータファイルを作成します。ファイルには2列が含まれ、それぞれ都市IDと都市名を表しています。以下のようになります：

      ```Plain
      200,'北京'
      ```

2. StarRocksのデータベース`test_db`にStarRocksテーブルを作成します。

   > **説明**
   >
   > 2.5.7バージョン以降、StarRocksはテーブル作成とパーティション追加時に自動的にバケット数（BUCKETS）を設定することをサポートしており、手動でバケット数を設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#确定分桶数量)を参照してください。

   a. `table1`という名前のプライマリキーモデルのテーブルを作成します。テーブルには`id`、`name`、`score`の3列が含まれ、それぞれユーザーID、ユーザー名、ユーザースコアを表し、プライマリキーは`id`列です。以下のようになります：

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
      ```

   b. `table2`という名前のプライマリキーモデルのテーブルを作成します。テーブルには`id`と`city`の2列が含まれ、それぞれ都市IDと都市名を表し、プライマリキーは`id`列です。以下のようになります：

      ```SQL
      CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "都市ID",
          `city` varchar(65533) NULL DEFAULT "" COMMENT "都市名"
      )
          ENGINE=OLAP
          PRIMARY KEY(`id`)
          DISTRIBUTED BY HASH(`id`);
   ```

3. 作成したデータファイル`file1.csv`と`file2.csv`をそれぞれHDFSクラスターの`/user/starrocks/`パス、AWS S3の`bucket_s3`ストレージスペースの`input`フォルダ、Google GCSの`bucket_gcs`ストレージスペースの`input`フォルダ、アリババクラウドOSSの`bucket_oss`ストレージスペースの`input`フォルダ、テンセントクラウドCOSの`bucket_cos`ストレージスペースの`input`フォルダ、ファーウェイクラウドOBSの`bucket_obs`ストレージスペースの`input`フォルダ、およびMinIOなどのS3プロトコルと互換性のあるその他のオブジェクトストレージの`bucket_minio`ストレージスペースの`input`フォルダ、およびAzure Storageの指定されたパスにアップロードします。

#### HDFSからのインポート

以下のステートメントを使用して、HDFSクラスターの`/user/starrocks/`パスにあるCSVファイル`file1.csv`と`file2.csv`をそれぞれStarRocksのテーブル`table1`と`table2`にインポートできます：

```SQL
LOAD LABEL test_db.label1
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
   (id, city)
)
WITH BROKER
(
    StorageCredentialParams
)
PROPERTIES
(
    "timeout" = "3600"
);
```


以上の例で、`StorageCredentialParams`は認証パラメータのセットを表しており、具体的にどのようなパラメータが含まれるかは、使用する認証方式によって異なります。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs) を参照してください。

バージョン3.1から、StarRocksはINSERT文とTABLEキーワードを使用して、HDFSからParquetまたはORC形式のデータファイルを直接インポートすることをサポートし、外部テーブルを事前に作成する手間を省きました。詳細は [INSERT文を使用したデータのインポート > TABLEキーワードを使用して外部データファイルを直接インポートする](../loading/InsertInto.md#通过-insert-into-select-以及表函数-files-导入外部数据文件) を参照してください。

#### AWS S3からのインポート

以下のステートメントを使用して、AWS S3のストレージスペース `bucket_s3` 内の `input` フォルダにあるCSVファイル `file1.csv` と `file2.csv` をそれぞれStarRocksのテーブル `table1` と `table2` にインポートできます：

```SQL
LOAD LABEL test_db.label2
(
    DATA INFILE("s3a://bucket_s3/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("s3a://bucket_s3/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

> **説明**
>
> Broker LoadはS3Aプロトコルを介してのみAWS S3にアクセスをサポートしているため、AWS S3からデータをインポートする際には、`DATA INFILE`で指定する対象ファイルのS3 URIのプレフィックスを `s3://` から `s3a://` に変更する必要があります。

上記の例で、`StorageCredentialParams`は認証パラメータのセットを表しており、具体的にどのようなパラメータが含まれるかは、使用する認証方式によって異なります。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3) を参照してください。

バージョン3.1から、StarRocksはINSERT文とTABLEキーワードを使用して、AWS S3からParquetまたはORC形式のデータファイルを直接インポートすることをサポートし、外部テーブルを事前に作成する手間を省きました。詳細は [INSERT文を使用したデータのインポート > TABLEキーワードを使用して外部データファイルを直接インポートする](../loading/InsertInto.md#通过-insert-into-select-以及表函数-files-导入外部数据文件) を参照してください。

#### Google GCSからのインポート

以下のステートメントを使用して、Google GCSのストレージスペース `bucket_gcs` 内の `input` フォルダにあるCSVファイル `file1.csv` と `file2.csv` をそれぞれStarRocksのテーブル `table1` と `table2` にインポートできます：

```SQL
LOAD LABEL test_db.label3
(
    DATA INFILE("gs://bucket_gcs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("gs://bucket_gcs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

> **説明**
>
> Broker Loadはgsプロトコルを介してのみGoogle GCSにアクセスをサポートしているため、Google GCSからデータをインポートする際には、ファイルパスで指定する対象ファイルのGCS URIが `gs://` をプレフィックスとして使用していることを確認する必要があります。

上記の例で、`StorageCredentialParams`は認証パラメータのセットを表しており、具体的にどのようなパラメータが含まれるかは、使用する認証方式によって異なります。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#google-gcs) を参照してください。

#### 阿里云OSSからのインポート

以下のステートメントを使用して、阿里云OSSのストレージスペース `bucket_oss` 内の `input` フォルダにあるCSVファイル `file1.csv` と `file2.csv` をそれぞれStarRocksのテーブル `table1` と `table2` にインポートできます：

```SQL
LOAD LABEL test_db.label4
(
    DATA INFILE("oss://bucket_oss/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("oss://bucket_oss/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

上記の例で、`StorageCredentialParams`は認証パラメータのセットを表しており、具体的にどのようなパラメータが含まれるかは、使用する認証方式によって異なります。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#阿里云-oss) を参照してください。

#### 腾讯云COSからのインポート

以下のステートメントを使用して、腾讯云COSのストレージスペース `bucket_cos` 内の `input` フォルダにあるCSVファイル `file1.csv` と `file2.csv` をそれぞれStarRocksのテーブル `table1` と `table2` にインポートできます：

```SQL
LOAD LABEL test_db.label5
(
    DATA INFILE("cosn://bucket_cos/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("cosn://bucket_cos/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

上記の例で、`StorageCredentialParams`は認証パラメータのセットを表しており、具体的にどのようなパラメータが含まれるかは、使用する認証方式によって異なります。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#腾讯云-cos) を参照してください。

#### 华为云OBSからのインポート

以下のステートメントを使用して、华为云OBSのストレージスペース `bucket_obs` 内の `input` フォルダにあるCSVファイル `file1.csv` と `file2.csv` をそれぞれStarRocksのテーブル `table1` と `table2` にインポートできます：

```SQL
LOAD LABEL test_db.label6
(
    DATA INFILE("obs://bucket_obs/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("obs://bucket_obs/input/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

> **説明**
>
> 华为云OBSからデータをインポートする際には、まず[依存ライブラリ](https://github.com/huaweicloud/obsa-hdfs/releases/download/v45/hadoop-huaweicloud-2.8.3-hw-45.jar)をダウンロードして **$BROKER_HOME/lib/** ディレクトリに追加し、Brokerを再起動する必要があります。

上記の例で、`StorageCredentialParams`は認証パラメータのセットを表しており、具体的にどのようなパラメータが含まれるかは、使用する認証方式によって異なります。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#华为云-obs) を参照してください。

#### S3プロトコル互換の他のオブジェクトストレージからのインポート

以下のステートメントを使用して、S3プロトコル互換のオブジェクトストレージ（例えばMinIO）の `bucket_minio` 内の `input` フォルダにあるCSVファイル `file1.csv` と `file2.csv` をそれぞれStarRocksのテーブル `table1` と `table2` にインポートできます：

```SQL
LOAD LABEL test_db.label7
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
    StorageCredentialParams
);
```

上記の例で、`StorageCredentialParams`は認証パラメータのセットを表しており、具体的にどのようなパラメータが含まれるかは、使用する認証方式によって異なります。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#其他兼容-s3-协议的对象存储) を参照してください。

#### Microsoft Azure Storageからのインポート

以下のステートメントを使用して、Azure Storageの指定されたパスにあるCSVファイル `file1.csv` と `file2.csv` をそれぞれStarRocksのテーブル `table1` と `table2` にインポートできます：

```SQL
LOAD LABEL test_db.label8
(
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    (id, name, score)
    ,
    DATA INFILE("wasb[s]://<container>@<storage_account>.blob.core.windows.net/<path>/file2.csv")
    INTO TABLE table2
    COLUMNS TERMINATED BY ","
    (id, city)
)
WITH BROKER
(
    StorageCredentialParams
);
```

> **注意**
  >
  > Azure Storageからデータをインポートする際には、使用するアクセスプロトコルとストレージサービスに基づいてファイルパスのプレフィックスを決定する必要があります。上記の例はBlob Storageを例にしています。
  >
  > - Blob Storageからデータをインポートする場合、使用するアクセスプロトコルに応じてファイルパスに `wasb://` または `wasbs://` をプレフィックスとして追加する必要があります：
  >   - HTTPプロトコルを使用してアクセスする場合は `wasb://` を、例えば `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*` として使用してください。
  >   - HTTPSプロトコルを使用してアクセスする場合は `wasbs://` を、例えば `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*` として使用してください。
  > - Azure Data Lake Storage Gen1 からデータをインポートする場合は、ファイルパスに `adl://` をプレフィックスとして追加する必要があります。例：`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
  > - Data Lake Storage Gen2 からデータをインポートする場合は、使用するアクセスプロトコルに応じてファイルパスに `abfs://` または `abfss://` をプレフィックスとして追加します：
  >   - HTTP プロトコルを使用してアクセスする場合は、`abfs://` をプレフィックスとして使用します。例：`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。
  >   - HTTPS プロトコルを使用してアクセスする場合は、`abfss://` をプレフィックスとして使用します。例：`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

上記の例で、`StorageCredentialParams` は認証パラメータのセットを表し、具体的にどのパラメータが含まれるかは、使用する認証方法によって異なります。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#microsoft-azure-storage) を参照してください。

#### データのクエリ

HDFS、AWS S3、Google GCS、アリババクラウド OSS、テンセントクラウド COS、またはファーウェイクラウド OBS からのインポートが完了した後、SELECT ステートメントを使用して StarRocks のテーブルデータを確認し、データが正常にインポートされたことを検証できます。

1. `table1` のデータをクエリするには、以下のようにします：

   ```SQL
   SELECT * FROM table1;
   +------+-------+-------+
   | id   | name  | score |
   +------+-------+-------+
   |    1 | Lily  |    23 |
   |    2 | Rose  |    23 |
   |    3 | Alice |    24 |
   |    4 | Julia |    25 |
   +------+-------+-------+
   4 rows in set (0.00 sec)
   ```

2. `table2` のデータをクエリするには、以下のようにします：

   ```SQL
   SELECT * FROM table2;
   +------+--------+
   | id   | city   |
   +------+--------+
   | 200  | 北京    |
   +------+--------+
   4 rows in set (0.01 sec)
   ```

### シングルテーブルインポート（Single-Table Load）ジョブの作成

データファイル1つ、または特定のパスにあるすべてのデータファイルをターゲットテーブルにインポートすることもできます。ここでは、AWS S3 の `bucket_s3` ストレージスペースの `input` フォルダに複数のデータファイルが含まれており、その中の1つのデータファイル名が `file1.csv` であると仮定します。これらのデータファイルはターゲットテーブル `table1` の列数と同じであり、列は順番にターゲットテーブル `table1` の列に対応しています。

データファイル `file1.csv` をターゲットテーブル `table1` にインポートするには、以下のステートメントを実行します:

```SQL
LOAD LABEL test_db.label_7
(
    DATA INFILE("s3a://bucket_s3/input/file1.csv")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    FORMAT AS "CSV"
)
WITH BROKER 
(
    StorageCredentialParams
)；
```

`input` フォルダのすべてのデータファイルをターゲットテーブル `table1` にインポートするには、以下のステートメントを実行します:

```SQL
LOAD LABEL test_db.label_8
(
    DATA INFILE("s3a://bucket_s3/input/*")
    INTO TABLE table1
    COLUMNS TERMINATED BY ","
    FORMAT AS "CSV"
)
WITH BROKER 
(
    StorageCredentialParams
)；
```

上記の2つの例で、`StorageCredentialParams` は認証パラメータのセットを表し、具体的にどのパラメータが含まれるかは、使用する認証方法によって異なります。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3) を参照してください。

### インポートジョブの確認

Broker Load では、SHOW LOAD ステートメントと curl コマンドの2つの方法でインポートジョブの実行状況を確認できます。

#### SHOW LOAD ステートメントの使用

[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) を参照してください。

#### curl コマンドの使用

コマンドの構文は以下の通りです：

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/<database_name>/_load_info?label=<label_name>'
```

> **注記**
>
> アカウントにパスワードが設定されていない場合は、`<username>:` のみを入力する必要があります。

例えば、以下のコマンドで `db1` データベースの `label1` というラベルのインポートジョブの実行状況を確認できます：

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/db1/_load_info?label=label1'
```

コマンドを実行すると、インポートジョブの結果情報 `jobInfo` が JSON 形式で返されます。以下のように表示されます：

```JSON
{"jobInfo":{"dbName":"default_cluster:db1","tblNames":["table1_simple"],"label":"label1","state":"FINISHED","failMsg":"","trackingUrl":""},"status":"OK","msg":"Success"}%
```

`jobInfo` には以下のパラメータが含まれます：

| **パラメータ**    | **説明**                                                     |
| ----------- | ------------------------------------------------------------ |
| dbName      | ターゲット StarRocks テーブルがあるデータベースの名前。                               |
| tblNames    | ターゲット StarRocks テーブルの名前。                        |
| label       | インポートジョブのラベル。                                             |
| state       | インポートジョブの状態。以下を含みます：<ul><li>`PENDING`：インポートジョブが実行待ちです。</li><li>`QUEUEING`：インポートジョブが実行待ちです。</li><li>`LOADING`：インポートジョブが実行中です。</li><li>`PREPARED`：トランザクションがコミットされました。</li><li>`FINISHED`：インポートジョブが成功しました。</li><li>`CANCELLED`：インポートジョブが失敗しました。</li></ul>[非同期インポート](./Loading_intro.md#非同期インポート)を参照してください。 |
| failMsg     | インポートジョブの失敗理由。インポートジョブの状態が`PENDING`、`LOADING`、または`FINISHED`の場合、このパラメータの値は`NULL`です。インポートジョブの状態が`CANCELLED`の場合、このパラメータの値には `type` と `msg` の2部分が含まれます：<ul><li>`type` には以下の値があります：</li><ul><li>`USER_CANCEL`：インポートジョブが手動でキャンセルされました。</li><li>`ETL_SUBMIT_FAIL`：インポートタスクの提出に失敗しました。</li><li>`ETL-QUALITY-UNSATISFIED`：データ品質が不十分で、インポートジョブのエラーデータ率が `max-filter-ratio` を超えました。</li><li>`LOAD-RUN-FAIL`：`LOADING` 状態でインポートジョブが失敗しました。</li><li>`TIMEOUT`：インポートジョブが許可されたタイムアウト時間内に完了しませんでした。</li><li>`UNKNOWN`：不明なインポートエラーです。</li></ul><li>`msg` は失敗理由の詳細情報を表示します。</li></ul> |
| trackingUrl | 品質が不十分なデータのアクセスアドレス。`curl` コマンドまたは `wget` コマンドを使用してアクセスできます。インポートジョブに品質が不十分なデータがない場合は、空の値が返されます。 |
| status      | インポートリクエストの状態。`OK` と `Fail` を含みます。                        |
| msg         | HTTP リクエストのエラーメッセージ。                                        |

### インポートジョブのキャンセル

インポートジョブの状態が **CANCELLED** または **FINISHED** でない場合、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) ステートメントを使用してインポートジョブをキャンセルできます。

例えば、以下のステートメントで `db1` データベースの `label1` ラベルのインポートジョブを取り消すことができます：

```SQL
CANCEL LOAD
FROM db1
WHERE LABEL = "label";
```

## ジョブの分割と並行実行

Broker Load ジョブは、1つまたは複数のサブタスクに分割されて並行して処理されます。ジョブのすべてのサブタスクは、トランザクションとして全体的に成功または失敗します。ジョブの分割は `LOAD LABEL` ステートメントの `data_desc` パラメータを使用して指定されます：

- 複数の `data_desc` パラメータが異なるテーブルへのインポートに対応している場合、各テーブルのデータインポートは1つのサブタスクに分割されます。

- 複数の `data_desc` パラメータが同じテーブルの異なるパーティションへのインポートに対応している場合、各パーティションのデータインポートは1つのサブタスクに分割されます。

各サブタスクはさらに複数のインスタンスに分割され、これらのインスタンスは BE 上で均等に分散されて並行して実行されます。インスタンスの分割は以下の [FE 設定](../administration/FE_configuration.md)によって決定されます：

- `min_bytes_per_broker_scanner`：単一インスタンスが処理する最小データ量。デフォルトは 64 MB です。

- `load_parallel_instance_num`：単一 BE でのジョブごとの並行インスタンス数。デフォルトは 1 です。バージョン 3.1 以降では使用されません。

   以下の式を使用して、単一サブタスクのインスタンスの総数を計算できます：

   単一サブタスクのインスタンスの総数 = min（サブタスクのインポートデータ量の合計 / `min_bytes_per_broker_scanner`、`load_parallel_instance_num` x BE の総数）

通常、インポートジョブには1つの `data_desc` のみがあり、1つのサブタスクに分割され、サブタスクは BE の総数と同じ数のインスタンスに分割されます。

## 関連設定項目

[FE 設定項目](../administration/FE_configuration.md) `max_broker_load_job_concurrency` は、StarRocks クラスターで並行して実行できる Broker Load ジョブの最大数を指定します。

StarRocks v2.4 以前のバージョンでは、ある時点で提出された Broker Load ジョブの総数が最大数を超えた場合、超過したジョブは提出時間に基づいてキューに入れられ、スケジュール待ちとなります。

StarRocks v2.5 版本では、ある時間内に提出された Broker Load ジョブの総数が最大数を超えた場合、超過したジョブは作成時に指定された優先度に従ってキューに入れられ、スケジュールを待ちます。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#opt_properties) のドキュメントのオプションパラメータ `priority` を参照してください。**QUEUEING** 状態または **LOADING** 状態の Broker Load ジョブの優先度を変更するには、[ALTER LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md) ステートメントを使用できます。

## よくある質問

[Broker Load のよくある質問](../faq/loading/Broker_load_faq.md) をご覧ください。
