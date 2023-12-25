---
displayed_sidebar: English
---

# HDFSまたはクラウドストレージからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、HDFSまたはクラウドストレージからStarRocksに大量のデータをロードするためのMySQLベースのBroker Loadというロード方法を提供しています。

Broker Loadは非同期ローディングモードで実行されます。ロードジョブを送信した後、StarRocksはそのジョブを非同期的に実行します。ジョブの結果を確認するには、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) ステートメントまたは `curl` コマンドを使用する必要があります。

Broker Loadは単一テーブルロードと複数テーブルロードをサポートしています。1つのBroker Loadジョブを実行することで、1つまたは複数のデータファイルを1つまたは複数の宛先テーブルにロードすることができます。Broker Loadは、複数のデータファイルをロードする各ロードジョブのトランザクションの原子性を保証します。原子性とは、1つのロードジョブで複数のデータファイルをロードする場合、すべてが成功するか、またはすべてが失敗することを意味します。一部のデータファイルのロードが成功し、他のファイルのロードが失敗することはありません。

Broker Loadはデータロード時のデータ変換をサポートし、データロード中にUPSERTおよびDELETE操作によるデータ変更もサポートしています。詳細は[ロード時のデータ変換](../loading/Etl_in_loading.md)および[ロードを通じたデータ変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

<InsertPrivNote />

## 背景情報

v2.4以前のStarRocksでは、Broker Loadジョブを実行する際に、StarRocksクラスタと外部ストレージシステム間の接続を設定するためにブローカーに依存していました。そのため、ロードステートメントで使用するブローカーを指定するために `WITH BROKER "<broker_name>"` を入力する必要がありました。これは「ブローカーベースのローディング」と呼ばれます。ブローカーは、ファイルシステムインターフェースと統合された独立したステートレスサービスです。ブローカーを使用すると、StarRocksは外部ストレージシステムに保存されているデータファイルにアクセスして読み取り、それらのデータファイルのデータを前処理してロードするための自身のコンピューティングリソースを使用できます。

v2.5以降、StarRocksはBroker Loadジョブを実行する際に、StarRocksクラスタと外部ストレージシステム間の接続を設定するためにブローカーに依存しなくなりました。そのため、ロードステートメントでブローカーを指定する必要はなくなりましたが、`WITH BROKER` キーワードは引き続き必要です。これは「ブローカーフリーローディング」と呼ばれます。

データがHDFSに保存されている場合、ブローカーフリーローディングが機能しない状況に遭遇することがあります。これは、データが複数のHDFSクラスタにまたがって保存されている場合や、複数のKerberosユーザーが設定されている場合に発生する可能性があります。このような状況では、ブローカーベースのローディングを使用することができます。これを成功させるには、少なくとも1つの独立したブローカーグループがデプロイされていることを確認してください。このような状況で認証設定とHA設定を指定する方法については、[HDFS](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。

> **注記**
>
> [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) ステートメントを使用して、StarRocksクラスタにデプロイされているブローカーを確認できます。ブローカーがデプロイされていない場合は、[ブローカーのデプロイ](../deployment/deploy_broker.md)に記載されている手順に従ってブローカーをデプロイできます。

## サポートされているデータファイル形式

Broker Loadは、以下のデータファイル形式をサポートしています：

- CSV

- Parquet

- ORC

> **注記**
>
> CSVデータについては、以下の点に注意してください：
>
> - コンマ(,)、タブ、パイプ(|)など、50バイトを超えないUTF-8文字列をテキスト区切り文字として使用できます。
> - Null値は`\N`を使用して表されます。例えば、データファイルが3つの列で構成されており、そのデータファイルのレコードが1列目と3列目にデータを保持しているが2列目にはデータがない場合、2列目に`\N`を使用してnull値を示す必要があります。つまり、レコードは`a,\N,b`としてコンパイルされる必要があり、`a,,b`ではなく、`a,,b`は2列目のレコードが空文字列を保持していることを示します。

## サポートされているストレージシステム

Broker Loadは、以下のストレージシステムをサポートしています：

- HDFS

- AWS S3

- Google GCS

- MinIOなどのその他のS3互換ストレージシステム

- Microsoft Azure Storage

## 仕組み

ロードジョブをFEに送信すると、FEはクエリプランを生成し、利用可能なBEの数とロードしたいデータファイルのサイズに基づいてクエリプランを分割し、各BEにクエリプランの一部を割り当てます。ロード中、各BEはHDFSまたはクラウドストレージシステムからデータファイルのデータを取得し、データを前処理してからStarRocksクラスタにロードします。すべてのBEがクエリプランの部分を完了した後、FEはロードジョブが成功したかどうかを判断します。

以下の図は、Broker Loadジョブのワークフローを示しています。

![Broker Loadのワークフロー](../assets/broker_load_how-to-work_en.png)

## 基本操作

### 複数テーブルロードジョブを作成する

このトピックでは、CSVを例に、複数のデータファイルを複数のテーブルにロードする方法について説明します。他のファイル形式でデータをロードする方法やBroker Loadの構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

StarRocksでは、一部のリテラルがSQL言語によって予約キーワードとして使用されています。これらのキーワードをSQLステートメントで直接使用しないでください。このようなキーワードをSQLステートメントで使用する場合は、バッククォート(`)のペアで囲んでください。詳細は[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データ例

1. ローカルファイルシステムにCSVファイルを作成します。

   a. `file1.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、ユーザースコアを順に表す3つの列で構成されています。

      ```Plain
      1,Lily,23
      2,Rose,23
      3,Alice,24
      4,Julia,25
      ```

   b. `file2.csv`という名前のCSVファイルを作成します。このファイルは、都市IDと都市名を順に表す2つの列で構成されています。

      ```Plain
      200,'Beijing'
      ```

2. StarRocksデータベース`test_db`にStarRocksテーブルを作成します。

   > **注記**
   >
   > v2.5.7以降、StarRocksはテーブルを作成する際やパーティションを追加する際に、バケット数(BUCKETS)を自動的に設定することができます。バケット数を手動で設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   a. `table1`という名前のPrimary Keyテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成され、`id`がプライマリキーです。

      ```SQL
      CREATE TABLE `table1`
      (
          `id` int(11) NOT NULL COMMENT "user ID",
          `name` varchar(65533) NULL DEFAULT "" COMMENT "user name",
          `score` int(11) NOT NULL DEFAULT "0" COMMENT "user score"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`) BUCKETS 10;
      ```

   b. `table2` という名前のプライマリキーを持つテーブルを作成します。このテーブルは2つのカラム、`id` と `city` で構成され、`id` がプライマリキーです。

      ```SQL
      CREATE TABLE `table2`
      (
          `id` int(11) NOT NULL COMMENT "city ID",
          `city` varchar(65533) NULL DEFAULT "" COMMENT "city name"
      )
      ENGINE=OLAP
      PRIMARY KEY(`id`)
      DISTRIBUTED BY HASH(`id`);
      ```

3. `file1.csv` と `file2.csv` をHDFSクラスターの `/user/starrocks/` パス、AWS S3バケット `bucket_s3` の `input` フォルダ、Google GCSバケット `bucket_gcs` の `input` フォルダ、MinIOバケット `bucket_minio` の `input` フォルダ、およびAzure Storageの指定されたパスにアップロードします。

#### HDFSからデータをロード

以下のステートメントを実行して、HDFSクラスターの `/user/starrocks` パスから `file1.csv` と `file2.csv` をそれぞれ `table1` と `table2` にロードします：

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

上記の例で、`StorageCredentialParams` は選択した認証方法に応じて異なる認証パラメータのグループを表します。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs) を参照してください。

#### AWS S3からデータをロード

以下のステートメントを実行して、AWS S3バケット `bucket_s3` の `input` フォルダから `file1.csv` と `file2.csv` をそれぞれ `table1` と `table2` にロードします：

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

> **注記**
>
> Broker Loadは、S3Aプロトコルに従ってのみAWS S3へのアクセスをサポートしています。したがって、AWS S3からデータをロードする際には、ファイルパスとして渡すS3 URIの `s3://` を `s3a://` に置き換える必要があります。

上記の例で、`StorageCredentialParams` は選択した認証方法に応じて異なる認証パラメータのグループを表します。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3) を参照してください。

v3.1以降、StarRocksはINSERTコマンドとTABLEキーワードを使用してAWS S3からParquet形式またはORC形式のファイルデータを直接ロードすることをサポートしており、外部テーブルを最初に作成する手間を省けます。詳細は [INSERTを使用したデータのロード > TABLEキーワードを使用して外部ソースのファイルから直接データを挿入する](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-files) を参照してください。

#### Google GCSからデータをロード

以下のステートメントを実行して、Google GCSバケット `bucket_gcs` の `input` フォルダから `file1.csv` と `file2.csv` をそれぞれ `table1` と `table2` にロードします：

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

> **注記**
>
> Broker Loadは、gsプロトコルに従ってのみGoogle GCSへのアクセスをサポートしています。そのため、Google GCSからデータをロードする際には、ファイルパスとして渡すGCS URIに `gs://` をプレフィックスとして含める必要があります。

上記の例で、`StorageCredentialParams` は選択した認証方法に応じて異なる認証パラメータのグループを表します。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#google-gcs) を参照してください。

#### 他のS3互換ストレージシステムからデータをロード

MinIOを例に挙げます。以下のステートメントを実行して、MinIOバケット `bucket_minio` の `input` フォルダから `file1.csv` と `file2.csv` をそれぞれ `table1` と `table2` にロードできます：

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

上記の例で、`StorageCredentialParams` は選択した認証方法に応じて異なる認証パラメータのグループを表します。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#other-s3-compatible-storage-system) を参照してください。

#### Microsoft Azure Storageからデータをロード

以下のステートメントを実行して、指定されたパスからAzure Storageの `file1.csv` と `file2.csv` をロードします：

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

> **通知**
  >
  > Azure Storageからデータをロードする際には、使用するアクセスプロトコルと特定のストレージサービスに基づいて、使用するプレフィックスを決定する必要があります。前の例では、Blob Storageを使用しています。
  >
  > - Blob Storageからデータをロードする場合は、ストレージアカウントへのアクセスに使用されるプロトコルに基づいて、ファイルパスに `wasb://` または `wasbs://` をプレフィックスとして含める必要があります：
  >   - Blob StorageがHTTPを通じてのみアクセスを許可している場合は、`wasb://` をプレフィックスとして使用します（例：`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`）。
  >   - Blob StorageがHTTPSを通じてのみアクセスを許可している場合は、`wasbs://` をプレフィックスとして使用します（例：`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`）。
  > - Data Lake Storage Gen1からデータをロードする場合は、ファイルパスに `adl://` をプレフィックスとして含める必要があります（例：`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`）。
  > - Data Lake Storage Gen2からデータをロードする場合は、ストレージアカウントへのアクセスに使用されるプロトコルに基づいて、ファイルパスに `abfs://` または `abfss://` をプレフィックスとして含める必要があります：
  >   - Data Lake Storage Gen2がHTTPを通じてのみアクセスを許可している場合は、`abfs://` をプレフィックスとして使用します（例：`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`）。
  >   - Data Lake Storage Gen2がHTTPSを通じてのみアクセスを許可している場合は、`abfss://` をプレフィックスとして使用します（例：`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`）。

上記の例で、`StorageCredentialParams` は選択した認証方法に応じて異なる認証パラメータのグループを表します。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#microsoft-azure-storage) を参照してください。

#### データのクエリ

HDFSクラスタ、AWS S3バケット、またはGoogle GCSバケットからのデータロードが完了したら、SELECTステートメントを使用してStarRocksテーブルのデータをクエリし、ロードが成功したことを確認できます。

1. 以下のステートメントを実行して `table1` のデータをクエリします：

   ```SQL
   MySQL [test_db]> SELECT * FROM table1;
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

2. 以下のステートメントを実行して `table2` のデータをクエリします：

   ```SQL
   MySQL [test_db]> SELECT * FROM table2;
   +------+--------+
   | id   | city   |
   +------+--------+
   | 200  | Beijing|
   +------+--------+
   1 row in set (0.01 sec)
   ```

### 単一テーブルロードジョブを作成する

また、指定したパスから単一のデータファイルまたはすべてのデータファイルを単一の宛先テーブルにロードすることもできます。AWS S3バケット `bucket_s3` には `input` という名前のフォルダが含まれており、この `input` フォルダには複数のデータファイルが含まれています。その中の1つは `file1.csv` という名前です。これらのデータファイルは `table1` と同じ数のカラムを持ち、各データファイルのカラムは `table1` のカラムに一対一で順番にマッピングできます。

`file1.csv` を `table1` にロードするには、次のステートメントを実行します。

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
);
```

`input` フォルダからすべてのデータファイルを `table1` にロードするには、次のステートメントを実行します。

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
);
```

上記の例では、`StorageCredentialParams` は選択した認証方法に応じて異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3)を参照してください。

### ロードジョブの表示

Broker Loadでは、SHOW LOADステートメントまたは `curl` コマンドを使用してロードジョブを表示できます。

#### SHOW LOADの使用

詳細については、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を参照してください。

#### curlの使用

構文は以下の通りです。

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/<database_name>/_load_info?label=<label_name>'
```

> **注**
>
> パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力します。

例えば、次のコマンドを実行して、`test_db` データベース内のラベルが `label1` のロードジョブに関する情報を表示できます。

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/test_db/_load_info?label=label1'
```

`curl` コマンドはロードジョブに関する情報をJSONオブジェクト `jobInfo` として返します。

```JSON
{"jobInfo":{"dbName":"default_cluster:test_db","tblNames":["table1_simple"],"label":"label1","state":"FINISHED","failMsg":"","trackingUrl":""},"status":"OK","msg":"Success"}%
```

以下の表は `jobInfo` のパラメーターの説明です。

| **パラメーター** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| dbName        | データがロードされるデータベースの名前           |
| tblNames      | データがロードされるテーブルの名前。             |
| label         | ロードジョブのラベル。                                   |
| state         | ロードジョブの状態。有効な値:<ul><li>`PENDING`: ロードジョブはキューに入ってスケジュールされるのを待っています。</li><li>`QUEUEING`: ロードジョブはキューに入ってスケジュールされるのを待っています。</li><li>`LOADING`: ロードジョブが実行中です。</li><li>`PREPARED`: トランザクションがコミットされました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul>詳細については、「データロードの概要」の「非同期ロード」セクションを参照してください[Overview of data loading](../loading/Loading_intro.md)。 |
| failMsg       | ロードジョブが失敗した理由。`state` 値が `PENDING`、`LOADING`、または `FINISHED` の場合、`failMsg` パラメーターには `NULL` が返されます。`state` 値が `CANCELLED` の場合、`failMsg` パラメーターに返される値は `type` と `msg` の2部分から構成されます。<ul><li>`type` 部分には以下の値があります:</li><ul><li>`USER_CANCEL`: ロードジョブは手動でキャンセルされました。</li><li>`ETL_SUBMIT_FAIL`: ロードジョブの提出に失敗しました。</li><li>`ETL_QUALITY_UNSATISFIED`: 不適格なデータの割合が `max-filter-ratio` パラメーターの値を超えたため、ロードジョブが失敗しました。</li><li>`LOAD_RUN_FAIL`: ロードジョブが `LOADING` ステージで失敗しました。</li><li>`TIMEOUT`: ロードジョブが指定されたタイムアウト期間内に完了できませんでした。</li><li>`UNKNOWN`: 不明なエラーによりロードジョブが失敗しました。</li></ul><li>`msg` 部分はロード失敗の詳細な原因を提供します。</li></ul> |
| trackingUrl   | ロードジョブで検出された不適格なデータにアクセスするために使用されるURL。`curl` または `wget` コマンドを使用してURLにアクセスし、不適格なデータを取得できます。不適格なデータが検出されない場合、`trackingUrl` パラメーターには `NULL` が返されます。 |
| status        | ロードジョブのHTTPリクエストの状態。有効な値: `OK` と `Fail`。 |
| msg           | ロードジョブのHTTPリクエストのエラー情報。  |

### ロードジョブのキャンセル

ロードジョブが **CANCELLED** または **FINISHED** ステージにない場合、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) ステートメントを使用してジョブをキャンセルできます。

例えば、次のステートメントを実行して、`test_db` データベース内のラベルが `label1` のロードジョブをキャンセルできます。

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label1";
```

## ジョブの分割と並行実行

Broker Loadジョブは、同時に実行される1つ以上のタスクに分割できます。ロードジョブ内のタスクは、単一のトランザクション内で実行されます。これらは全て成功するか、または失敗します。StarRocksは、`LOAD` ステートメント内の `data_desc` の宣言に基づいて各ロードジョブを分割します。

- 複数の `data_desc` パラメーターを宣言し、それぞれが異なるテーブルを指定する場合、各テーブルのデータをロードするタスクが生成されます。

- 複数の `data_desc` パラメーターを宣言し、それぞれが同じテーブルの異なるパーティションを指定する場合、各パーティションのデータをロードするタスクが生成されます。

さらに、各タスクは1つ以上のインスタンスにさらに分割され、StarRocksクラスターのBEに均等に分散して並行して実行されます。StarRocksは、以下の[FE設定](../administration/FE_configuration.md#fe-configuration-items)に基づいて各タスクを分割します。

- `min_bytes_per_broker_scanner`: 各インスタンスが処理するデータの最小量。デフォルトは64MBです。

- `load_parallel_instance_num`: 個々のBEで各ロードジョブに許可される並行インスタンスの数。デフォルトは1です。
  
  個々のタスクのインスタンス数を計算するには、以下の式を使用します。

  **個々のタスクのインスタンス数 = min(タスクによってロードされるデータ量 / `min_bytes_per_broker_scanner`, `load_parallel_instance_num` x BEの数)**

ほとんどの場合、ロードジョブごとに1つの `data_desc` が宣言され、各ロードジョブは1つのタスクにのみ分割され、タスクはBEの数と同じ数のインスタンスに分割されます。

## 関連する設定項目

[FE設定項目](../administration/FE_configuration.md#fe-configuration-items)の `max_broker_load_job_concurrency` は、StarRocksクラスター内で同時に実行できるBroker Loadジョブの最大数を指定します。

StarRocks v2.4以前では、特定の期間内に送信されたBroker Loadジョブの総数が最大数を超えた場合、超過したジョブはキューに入れられ、送信時間に基づいてスケジュールされます。

StarRocks v2.5以降、特定の期間内に送信されたBroker Loadジョブの総数が最大数を超えた場合、超過したジョブはキューに入れられ、優先度に基づいてスケジュールされます。ジョブ作成時に`priority`パラメータを使用して、ジョブの優先度を指定することができます。詳細は[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#opt_properties)を参照してください。また、[ALTER LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md)を使用して、**QUEUEING**または**LOADING**状態にある既存のジョブの優先度を変更することもできます。
