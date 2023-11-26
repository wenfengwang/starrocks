---
displayed_sidebar: "Japanese"
---

# HDFSまたはクラウドストレージからデータを読み込む

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、MySQLベースのBroker Loadという読み込み方法を提供しており、これを使用してHDFSまたはクラウドストレージから大量のデータをStarRocksに読み込むことができます。

Broker Loadは非同期読み込みモードで実行されます。読み込みジョブを送信すると、StarRocksは非同期でジョブを実行します。ジョブの結果を確認するには、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)ステートメントまたは`curl`コマンドを使用する必要があります。

Broker Loadは、単一テーブルの読み込みと複数テーブルの読み込みの両方をサポートしています。1つのBroker Loadジョブを実行することで、1つまたは複数のデータファイルを1つまたは複数の宛先テーブルに読み込むことができます。Broker Loadは、複数のデータファイルを読み込むために実行される各読み込みジョブのトランザクション的な原子性を保証します。原子性とは、1つの読み込みジョブで複数のデータファイルを読み込む場合、すべてが成功するか、すべてが失敗するかのいずれかであることを意味します。つまり、一部のデータファイルの読み込みが成功し、他のファイルの読み込みが失敗することはありません。

Broker Loadは、データの読み込み時にデータ変換をサポートし、データの読み込み中にUPSERTおよびDELETE操作によるデータ変更をサポートしています。詳細については、[読み込み時のデータ変換](../loading/Etl_in_loading.md)および[読み込みを介したデータ変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

<InsertPrivNote />

## 背景情報

v2.4以前のバージョンでは、StarRocksはBroker Loadジョブを実行する際に、ブローカーを使用してStarRocksクラスタと外部ストレージシステムの間の接続を設定していました。そのため、ロードステートメントで使用するブローカーを指定するために`WITH BROKER "<broker_name>"`を入力する必要があります。これを「ブローカーベースの読み込み」と呼びます。ブローカーは、ファイルシステムインターフェースと統合された独立したステートレスサービスです。ブローカーを使用することで、StarRocksは外部ストレージシステムに格納されているデータファイルにアクセスし、データファイルのデータを自身の計算リソースを使用して前処理および読み込みすることができます。

v2.5以降、StarRocksはBroker Loadジョブを実行する際に、ブローカーを使用してStarRocksクラスタと外部ストレージシステムの間の接続を設定する必要がありません。そのため、ロードステートメントでブローカーを指定する必要はありませんが、`WITH BROKER`キーワードは保持する必要があります。これを「ブローカーフリーの読み込み」と呼びます。

データがHDFSに格納されている場合、ブローカーフリーの読み込みが機能しない場合があります。これは、データが複数のHDFSクラスタにまたがって格納されている場合や、複数のKerberosユーザーが構成されている場合に発生する可能性があります。このような場合は、代わりにブローカーベースの読み込みを使用することができます。これを成功させるためには、少なくとも1つの独立したブローカーグループが展開されていることを確認してください。これらの状況での認証構成およびHA構成の指定方法については、[HDFS](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。

> **注意**
>
> [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md)ステートメントを使用して、StarRocksクラスタに展開されているブローカーを確認できます。ブローカーが展開されていない場合は、[ブローカーの展開](../deployment/deploy_broker.md)に記載されている手順に従ってブローカーを展開できます。

## サポートされているデータファイル形式

Broker Loadは、次のデータファイル形式をサポートしています。

- CSV

- Parquet

- ORC

> **注意**
>
> CSVデータの場合、次の点に注意してください。
>
> - テキスト区切り記号として、50バイトを超えないUTF-8文字列（カンマ（,）、タブ、またはパイプ（|））を使用できます。
> - Null値は`\N`を使用して表します。たとえば、データファイルは3つの列から構成されており、そのデータファイルのレコードは、最初の列と3番目の列にデータを保持していますが、2番目の列にはデータがありません。この場合、2番目の列には`\N`を使用してヌル値を示す必要があります。つまり、レコードは`a,\N,b`とコンパイルする必要があります。`a,,b`は、レコードの2番目の列に空の文字列があることを示します。

## サポートされているストレージシステム

Broker Loadは、次のストレージシステムをサポートしています。

- HDFS

- AWS S3

- Google GCS

- MinIOなどの他のS3互換ストレージシステム

- Microsoft Azure Storage

## 動作原理

FEにロードジョブを送信すると、FEはクエリプランを生成し、利用可能なBEの数と読み込みたいデータファイルのサイズに基づいてクエリプランを分割し、それぞれのクエリプランの一部を利用可能なBEに割り当てます。読み込み中、関連する各BEは、HDFSまたはクラウドストレージシステムのデータファイルのデータを取得し、データを前処理し、データをStarRocksクラスタに読み込みます。すべてのBEがクエリプランの各部分を完了した後、FEはロードジョブが成功したかどうかを判断します。

次の図は、Broker Loadジョブのワークフローを示しています。

![Broker Loadのワークフロー](../assets/broker_load_how-to-work_en.png)

## 基本操作

### 複数テーブルの読み込みジョブを作成する

このトピックでは、CSVを使用して、複数のデータファイルを複数のテーブルに読み込む方法について説明します。他のファイル形式でデータを読み込む方法や、Broker Loadの構文およびパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

StarRocksでは、いくつかのリテラルがSQL言語によって予約されたキーワードとして使用されています。SQLステートメントでこのようなキーワードを直接使用しないでください。SQLステートメントでこのようなキーワードを使用する場合は、バッククォート（`）で囲んでください。詳細については、[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データの例

1. ローカルファイルシステムにCSVファイルを作成します。

   a. `file1.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、およびユーザースコアを順番に表す3つの列で構成されています。

      ```Plain
      1,Lily,23
      2,Rose,23
      3,Alice,24
      4,Julia,25
      ```

   b. `file2.csv`という名前のCSVファイルを作成します。このファイルは、都市IDと都市名を順番に表す2つの列で構成されています。

      ```Plain
      200,'Beijing'
      ```

2. StarRocksデータベース`test_db`にStarRocksテーブルを作成します。

   > **注意**
   >
   > v2.5.7以降、StarRocksはテーブルを作成するかパーティションを追加する際に、バケットの数（BUCKETS）を自動的に設定することができます。バケットの数を手動で設定する必要はありません。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   a. `table1`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、および`score`の3つの列で構成されており、`id`がプライマリキーです。

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
      ```

   b. `table2`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`と`city`の2つの列で構成されており、`id`がプライマリキーです。

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

3. `file1.csv`と`file2.csv`をHDFSクラスタの`/user/starrocks/`パス、AWS S3バケット`bucket_s3`の`input`フォルダ、Google GCSバケット`bucket_gcs`の`input`フォルダ、MinIOバケット`bucket_minio`の`input`フォルダ、およびAzure Storageの指定されたパスにアップロードします。

#### HDFSからデータを読み込む

次のステートメントを実行して、HDFSクラスタの`/user/starrocks`パスから`file1.csv`と`file2.csv`をそれぞれ`table1`と`table2`に読み込みます。

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

前の例では、`StorageCredentialParams`は、選択した認証方法によって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。

#### AWS S3からデータを読み込む

次のステートメントを実行して、AWS S3バケット`bucket_s3`の`input`フォルダから`file1.csv`と`file2.csv`をそれぞれ`table1`と`table2`に読み込みます。

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

> **注意**
>
> Broker Loadは、S3Aプロトコルに従ってAWS S3にのみアクセスすることができます。したがって、AWS S3からデータを読み込む場合は、S3 URIのファイルパスとして渡す`s3://`を`s3a://`に置き換える必要があります。

前の例では、`StorageCredentialParams`は、選択した認証方法によって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3)を参照してください。

v3.1以降、StarRocksは、INSERTコマンドとTABLEキーワードを使用して、AWS S3からParquet形式またはORC形式のファイルのデータを直接読み込むことができます。これにより、最初に外部テーブルを作成する手間が省けます。詳細については、[INSERTを使用したデータの読み込み > TABLEキーワードを使用して外部ソースのファイルからデータを直接挿入する](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-table-keyword)を参照してください。

#### Google GCSからデータを読み込む

次のステートメントを実行して、Google GCSバケット`bucket_gcs`の`input`フォルダから`file1.csv`と`file2.csv`をそれぞれ`table1`と`table2`に読み込みます。

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

> **注意**
>
> Broker Loadは、gsプロトコルに従ってGoogle GCSにのみアクセスすることができます。したがって、Google GCSからデータを読み込む場合は、ファイルパスとして渡すGCS URIに`gs://`を含める必要があります。

前の例では、`StorageCredentialParams`は、選択した認証方法によって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#google-gcs)を参照してください。

#### 他のS3互換ストレージシステムからデータを読み込む

MinIOを例に挙げます。次のステートメントを実行して、MinIOバケット`bucket_minio`の`input`フォルダから`file1.csv`と`file2.csv`をそれぞれ`table1`と`table2`に読み込みます。

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

前の例では、`StorageCredentialParams`は、選択した認証方法によって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#other-s3-compatible-storage-system)を参照してください。

#### Microsoft Azure Storageからデータを読み込む

指定されたパスから`file1.csv`と`file2.csv`を読み込むには、次のステートメントを実行します。

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
> Azure Storageからデータを読み込む場合、使用するアクセスプロトコルと具体的なストレージサービスに基づいて、どの接頭辞を使用するかを決定する必要があります。前の例では、Blob Storageを例に挙げています。
>
> - Blob Storageからデータを読み込む場合、ストレージアカウントへのアクセスがHTTP経由のみ許可されている場合は、ファイルパスに`wasb://`を含める必要があります。たとえば、`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`です。
> - Blob Storageからデータを読み込む場合、ストレージアカウントへのアクセスがHTTPS経由のみ許可されている場合は、ファイルパスに`wasbs://`を含める必要があります。たとえば、`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`です。
> - Data Lake Storage Gen1からデータを読み込む場合、ファイルパスに`adl://`を含める必要があります。たとえば、`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`です。
> - Data Lake Storage Gen2からデータを読み込む場合、ストレージアカウントへのアクセスがHTTP経由のみ許可されている場合は、ファイルパスに`abfs://`を含める必要があります。たとえば、`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`です。
> - Data Lake Storage Gen2からデータを読み込む場合、ストレージアカウントへのアクセスがHTTPS経由のみ許可されている場合は、ファイルパスに`abfss://`を含める必要があります。たとえば、`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`です。

前の例では、`StorageCredentialParams`は、選択した認証方法によって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#microsoft-azure-storage)を参照してください。

#### データのクエリ

HDFSクラスタ、AWS S3バケット、またはGoogle GCSバケットからのデータの読み込みが完了したら、SELECTステートメントを使用してStarRocksテーブルのデータをクエリし、読み込みが成功したことを確認できます。

1. `table1`のデータをクエリするには、次のステートメントを実行します。

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

1. `table2`のデータをクエリするには、次のステートメントを実行します。

   ```SQL
   MySQL [test_db]> SELECT * FROM table2;
   +------+--------+
   | id   | city   |
   +------+--------+
   | 200  | Beijing|
   +------+--------+
   4 rows in set (0.01 sec)
   ```

### 単一テーブルの読み込みジョブを作成する

単一のデータファイルまたは指定されたパスのすべてのデータファイルを単一の宛先テーブルに読み込むこともできます。AWS S3バケット`bucket_s3`に`input`という名前のフォルダが含まれていると仮定します。`input`フォルダには、`file1.csv`という名前のファイルを含む複数のデータファイルが含まれています。これらのデータファイルは、`table1`と同じ数の列で構成されており、これらのデータファイルの各列は`table1`の列と1対1で対応付けることができます。

`file1.csv`を`table1`に読み込むには、次のステートメントを実行します。

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

`input`フォルダからすべてのデータファイルを`table1`に読み込むには、次のステートメントを実行します。

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

前の例では、`StorageCredentialParams`は、選択した認証方法によって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3)を参照してください。

### ロードジョブを表示する

Broker Loadでは、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)ステートメントまたは`curl`コマンドを使用してロードジョブを表示することができます。

#### SHOW LOADを使用する

詳細については、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を参照してください。

#### curlを使用する

構文は次のとおりです。

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/<database_name>/_load_info?label=<label_name>'
```

> **注意**
>
> パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力する必要があります。

たとえば、次のコマンドを実行して、データベース`test_db`のラベルが`label1`であるロードジョブの情報を表示できます。

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/test_db/_load_info?label=label1'
```

`curl`コマンドは、JSONオブジェクト`jobInfo`としてロードジョブの情報を返します。

```JSON
{"jobInfo":{"dbName":"default_cluster:test_db","tblNames":["table1_simple"],"label":"label1","state":"FINISHED","failMsg":"","trackingUrl":""},"status":"OK","msg":"Success"}%
```

次の表は、`jobInfo`内のパラメータを説明しています。

| **パラメータ** | **説明**                                                     |
| ------------- | ------------------------------------------------------------ |
| dbName        | データが読み込まれるデータベースの名前                         |
| tblNames      | データが読み込まれるテーブルの名前                             |
| label         | ロードジョブのラベル                                           |
| state         | ロードジョブのステータス。有効な値:<ul><li>`PENDING`：キューに入ってスケジュール待ちのロードジョブです。</li><li>`QUEUEING`：キューに入ってスケジュール待ちのロードジョブです。</li><li>`LOADING`：ロードジョブが実行中です。</li><li>`PREPARED`：トランザクションがコミットされました。</li><li>`FINISHED`：ロードジョブが成功しました。</li><li>`CANCELLED`：ロードジョブが失敗しました。</li></ul>詳細については、[データの読み込みの概要](../loading/Loading_intro.md)の「非同期読み込み」セクションを参照してください。 |
| failMsg       | ロードジョブの失敗理由。ロードジョブの`state`値が`PENDING`、`LOADING`、または`FINISHED`の場合、`failMsg`パラメータには`NULL`が返されます。ロードジョブの`state`値が`CANCELLED`の場合、`failMsg`パラメータの値は`type`と`msg`の2つの部分で構成されます。<ul><li>`type`部分は、次のいずれかの値になります:</li><ul><li>`USER_CANCEL`：ロードジョブが手動でキャンセルされました。</li><li>`ETL_SUBMIT_FAIL`：ロードジョブの送信に失敗しました。</li><li>`ETL-QUALITY-UNSATISFIED`：ロードジョブが`max-filter-ratio`パラメータの値を超える不適格データの割合を持つため、失敗しました。</li><li>`LOAD-RUN-FAIL`：ロードジョブが`LOADING`ステージで失敗しました。</li><li>`TIMEOUT`：ロードジョブが指定されたタイムアウト期間内に完了しなかったため、失敗しました。</li><li>`UNKNOWN`：ロードジョブが不明なエラーにより失敗しました。</li></ul><li>`msg`部分は、ロードの失敗の詳細な原因を提供します。</li></ul> |
| trackingUrl   | ロードジョブで検出された不適格データにアクセスするために使用するURL。URLにアクセスして不適格データを取得するには、`curl`または`wget`コマンドを使用できます。不適格データが検出されない場合、`trackingUrl`パラメータには`NULL`が返されます。 |
| status        | ロードジョブのHTTPリクエストのステータス。有効な値: `OK`および`Fail` |
| msg           | ロードジョブのHTTPリクエストのエラー情報。                     |

### ロードジョブをキャンセルする

ロードジョブが**CANCELLED**または**FINISHED**のステージにない場合、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)ステートメントを使用してジョブをキャンセルすることができます。

たとえば、データベース`test_db`のラベルが`label1`であるロードジョブをキャンセルするには、次のステートメントを実行します。

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label";
```

## ジョブの分割と並行実行

Broker Loadジョブは、1つまたは複数のタスクに分割されて並行して実行されることがあります。ロードジョブ内のタスクは、単一のトランザクション内で実行されます。すべてのタスクは成功するか失敗するかのいずれかである必要があります。StarRocksは、`LOAD`ステートメントで`data_desc`を宣言する方法に基づいて、各ロードジョブを分割します。

- 異なるテーブルを指定する複数の`data_desc`パラメータを宣言する場合、各テーブルのデータを読み込むためのタスクが生成されます。

- 同じテーブルの異なるパーティションを指定する複数の`data_desc`パラメータを宣言する場合、各パーティションのデータを読み込むためのタスクが生成されます。

さらに、各タスクは1つ以上のインスタンスにさらに分割され、これらのインスタンスはStarRocksクラスタのBEに均等に分散されて並行して実行されます。StarRocksは、各タスクを次の[FEの設定](../administration/Configuration.md#fe-configuration-items)に基づいて分割します。

- `min_bytes_per_broker_scanner`：各インスタンスが処理する最小データ量。デフォルトは64 MBです。

- `load_parallel_instance_num`：各ロードジョブで個々のBEで許可される同時インスタンスの数。デフォルトは1です。
  
  個々のタスクのインスタンス数を計算するための次の式を使用できます。

  **個々のタスクのインスタンス数 = 個々のタスクで読み込むデータ量/`min_bytes_per_broker_scanner`の量 x BEの数 x `load_parallel_instance_num`**

ほとんどの場合、各ロードジョブには1つの`data_desc`が宣言され、各ロードジョブは1つのタスクにのみ分割され、タスクはBEの数と同じ数のインスタンスに分割されます。

## 関連する設定項目

[FEの設定項目](../administration/Configuration.md#fe-configuration-items)`max_broker_load_job_concurrency`は、StarRocksクラスタ内で同時に実行できるBroker Loadジョブの最大数を指定します。

StarRocks v2.4以前では、特定の期間内に送信されたBroker Loadジョブの総数が最大数を超える場合、余剰のジョブはキューに入れられ、送信時間に基づいてスケジュールされます。

StarRocks v2.5以降では、特定の期間内に送信されたBroker Loadジョブの総数が最大数を超える場合、余剰のジョブは優先度に基づいてキューに入れられ、スケジュールされます。ジョブの優先度は、ジョブ作成時に`priority`パラメータを使用して指定することができます。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#opt_properties)を参照してください。また、**QUEUEING**または**LOADING**ステートにある既存のジョブの優先度を変更するために[ALTER LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md)を使用することもできます。
