---
displayed_sidebar: "Japanese"
---

# HDFSまたはクラウドストレージからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、HDFSまたはクラウドストレージから大量のデータをStarRocksにロードするのをサポートするために、MySQLベースのBroker Loadというロードメソッドを提供しています。

Broker Loadは非同期ロードモードで実行されます。ロードジョブを提出すると、StarRocksは非同期でジョブを実行します。ロードしたジョブの結果を確認するには、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)ステートメントまたは`curl`コマンドを使用する必要があります。

Broker Loadは、単一テーブルのロードと複数テーブルのロードをサポートしています。1つのBroker Loadジョブを実行することで、1つまたは複数のデータファイルを1つまたは複数の宛先テーブルにロードすることができます。Broker Loadは、複数のデータファイルをロードするために実行される各ロードジョブのトランザクション的な原子性を保証します。原子性とは、1つのロードジョブで複数のデータファイルをロードする際に、すべてが成功するか、すべてが失敗するかのいずれかであることを意味します。1つのデータファイルのロードが成功し、他のファイルのロードが失敗することは決してありません。

Broker Loadは、データロード時のデータ変換をサポートし、データロード中のUPSERTおよびDELETE操作によるデータ変更をサポートしています。詳細については、[ロード時のデータ変換](../loading/Etl_in_loading.md)および[ロードによるデータ変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

<InsertPrivNote />

## 背景情報

v2.4以前では、StarRocksはBroker Loadジョブを実行する際にStarRocksクラスタと外部ストレージシステム間の接続を確立するためにブローカーに依存していました。したがって、ロードステートメントで使用するために、`WITH BROKER "<broker_name>"`を入力する必要がありました。これを "ブローカーベースのローディング" と呼びます。ブローカーは、独立したステートレスなサービスであり、ファイルシステムインターフェイスと統合されています。ブローカーを使用すると、StarRocksは外部ストレージシステムに保存されているデータファイルにアクセスして読み取ることができ、これらのデータファイルのデータを前処理し、ロードするために独自の計算リソースを使用できます。

v2.5以降では、StarRocksはブローカーに依存せずにBroker Loadジョブを実行する際にStarRocksクラスタと外部ストレージシステム間の接続を確立するためにブローカーに依存しません。そのため、ロードステートメントでブローカーを指定する必要はありませんが、`WITH BROKER`キーワードは引き続き保持する必要があります。これを "ブローカーフリー・ローディング" と呼びます。

データがHDFSに保存されている場合、ブローカーフリー・ローディングが機能しないことがあります。これは、データが複数のHDFSクラスタに保存されている場合や、複数のKerberosユーザが構成されている場合に発生する可能性があります。このような状況では、代わりにブローカーベースのローディングを使用することができます。これを成功させるためには、少なくとも1つの独立したブローカーグループが展開されていることを確認してください。これらの状況で認証構成とHA構成を指定する方法については、[HDFS](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。

> **注意**
>
> StarRocksクラスタに展開されているブローカーを確認するために、[SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md)ステートメントを使用できます。ブローカーが展開されていない場合は、[ブローカーを展開](../deployment/deploy_broker.md)する手順に従ってブローカーを展開できます。

## サポートされるデータファイル形式

Broker Loadは、次のデータファイル形式をサポートしています：

- CSV

- Parquet

- ORC

> **注意**
>
> CSVデータについては、次の点に注意してください：
>
> - 50バイトを超えないUTF-8文字列（カンマ（、）、タブ、またはパイプ（|）など）を文字区切り文字として使用することができます。
> - Null値は`\N`を使用して示します。たとえば、データファイルが3つの列で構成され、そのデータファイルのレコードが最初の列と三番目の列にデータを保持しており、二番目の列にはデータが含まれていない場合、この状況では、二番目の列にnull値を示すために`\N`を使用する必要があります。これは、レコードを`a,\N,b`として編成する必要があることを意味します。`a,,b`は、レコードの二番目の列に空の文字列が含まれていることを示します。

## サポートされるストレージシステム

Broker Loadは、次のストレージシステムをサポートしています：

- HDFS

- AWS S3

- Google GCS

- MinIOなどの他のS3互換ストレージシステム

- Microsoft Azure Storage

## 動作原理

FEにロードジョブを提出すると、FEはクエリプランを生成し、使用可能なBEの数とロードするデータファイルのサイズに基づいてクエリプランを分割し、それぞれのクエリプランを使用可能なBEに割り当てます。ロード中、関与する各BEは、HDFSまたはクラウドストレージシステムからデータファイルのデータを取得し、データを前処理し、その後データをStarRocksクラスタにロードします。すべてのBEがクエリプランのそれぞれの部分を完了した後、FEはロードジョブが成功したかどうかを判断します。

次の図は、ブローカーロードジョブのワークフローを示しています。

![ブローカーロードのワークフロー](../assets/broker_load_how-to-work_en.png)

## 基本操作

### 複数テーブルロードジョブを作成する

このトピックでは、CSVを使用して複数のデータファイルを複数のテーブルにロードする方法を説明します。他のファイル形式のデータをロードする方法や、Broker Loadの文法とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

StarRocksでは、一部のリテラルがSQL言語によって予約されているため、SQLステートメントでこれらの予約語を直接使用しないでください。SQLステートメントでそのようなキーワードを使用する場合は、バッククォート（`）で囲んでください。詳細については、[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データの例

1. ローカルファイルシステムにCSVファイルを作成します。

   a. `file1.csv`という名前のCSVファイルを作成します。ファイルは、ユーザーID、ユーザー名、およびユーザースコアを順に表す3つの列で構成されています。

      ```Plain
      1,Lily,23
      2,Rose,23
      3,Alice,24
      4,Julia,25
      ```

   b. `file2.csv`という名前のCSVファイルを作成します。ファイルは、都市IDと都市名を順に表す2つの列で構成されています。

      ```Plain
      200,'Beijing'
      ```

2. StarRocksデータベース `test_db` にStarRocksテーブルを作成します。

   > **注意**
   >
   > v2.5.7以降、StarRocksは、テーブルを作成したりパーティションを追加する際にバケツの数（BUCKETS）を自動的に設定できます。バケツの数は手動で設定する必要はもはやありません。詳細については、[バケツの数を決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   a. `table1` という名前のプライマリキー付きテーブルを作成します。このテーブルには、`id`、`name`、`score`の3つの列が含まれており、`id`がプライマリーキーです。

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

   b. `table2` という名前のプライマリキー付きテーブルを作成します。このテーブルには、`id`、`city`の2つの列が含まれており、`id`がプライマリーキーです。

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

3. `file1.csv`と`file2.csv`をHDFSクラスタの`/user/starrocks/`パス、AWS S3バケットの`bucket_s3`の`input`フォルダ、Google GCSバケットの`bucket_gcs`の`input`フォルダ、MinIOバケットの`bucket_minio`の`input`フォルダ、およびAzure Storageの指定されたパスにアップロードします。

#### HDFSからデータをロードする

以下のステートメントを実行して、HDFSクラスタの`/user/starrocks`パスから`file1.csv`および`file2.csv`をそれぞれ`table1`と`table2`にロードします。

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
前述の例では、`StorageCredentialParams` は、選択した認証メソッドによって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。

#### AWS S3 からデータを読み込む

次のステートメントを実行して、`file1.csv` と `file2.csv` を AWS S3 バケット `bucket_s3` の `input` フォルダからそれぞれ `table1` と `table2` に読み込みます。

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
> ブローカー ロードは、S3A プロトコルに基づく AWS S3 へのアクセスのみをサポートしています。したがって、AWS S3 からデータを読み込む際には、ファイルパスとして渡す S3 URI の `s3://` を `s3a://` に置換する必要があります。

前述の例では、`StorageCredentialParams` は、選択した認証メソッドによって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3)を参照してください。

v3.1 以降、StarRocks は、INSERT コマンドと TABLE キーワードを使用して AWS S3 から Parquet 形式または ORC 形式のファイルのデータを直接読み込むことをサポートしており、この方法により、まず外部テーブルを作成する手間が省けます。詳細については、[INSERT を使用したデータの読み込み > TABLE キーワードを使用して外部ソースのファイルからデータを直接挿入](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-table-keyword)を参照してください。

#### Google GCS からデータを読み込む

次のステートメントを実行して、`file1.csv` と `file2.csv` を Google GCS バケット `bucket_gcs` の `input` フォルダからそれぞれ `table1` と `table2` に読み込みます。

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
> ブローカー ロードは、gs プロトコルに基づく Google GCS へのアクセスのみをサポートしています。したがって、Google GCS からデータを読み込む際には、ファイルパスとして渡す GCS URI のプレフィックスとして `gs://` を含める必要があります。

前述の例では、`StorageCredentialParams` は、選択した認証メソッドによって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#google-gcs)を参照してください。

#### 他の S3 互換ストレージシステムからデータを読み込む

MinIO を例に挙げます。次のステートメントを実行して、MinIO バケット `bucket_minio` の `input` フォルダから `file1.csv` と `file2.csv` をそれぞれ `table1` と `table2` に読み込みます。

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

前述の例では、`StorageCredentialParams` は、選択した認証メソッドによって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#other-s3-compatible-storage-system)を参照してください。

#### Microsoft Azure Storage からデータを読み込む

指定したパスから Azure Storage の `file1.csv` と `file2.csv` を読み込む次のステートメントを実行します。

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
> Azure Storage からデータを読み込む場合は、アクセスプロトコルと使用する特定のストレージサービスに基づいて、どのプレフィックスを使用するかを決定する必要があります。前述の例では Blob Storage を例に挙げています。
>
> - Blob Storage からデータを読み込む際には、ストレージ アカウントへのアクセスが HTTP を介してのみ許可される場合は、ファイルパスにプレフィックスとして `wasb://` を含める必要があります。例: `wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
>   - Blob Storage からデータを読み込む際には、ストレージ アカウントへのアクセスが HTTPS を介してのみ許可される場合は、ファイルパスにプレフィックスとして `wasbs://` を含める必要があります。例: `wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`
> - Data Lake Storage Gen1 からデータを読み込む際には、ファイルパスにプレフィックスとして `adl://` を含める必要があります。例: `adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
> - Data Lake Storage Gen2 からデータを読み込む際には、ストレージ アカウントへのアクセスが HTTP を介してのみ許可される場合は、ファイルパスにプレフィックスとして `abfs://` を含める必要があります。例: `abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。ストレージ アカウントへのアクセスが HTTPS を介してのみ許可される場合は、ファイルパスにプレフィックスとして `abfss://` を含める必要があります。例: `abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

前述の例では、`StorageCredentialParams` は、選択した認証メソッドによって異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#microsoft-azure-storage)を参照してください。

#### データのクエリ

HDFS クラスタ、AWS S3 バケット、または Google GCS バケットからのデータの読み込みが完了したら、SELECT ステートメントを使用して StarRocks テーブルのデータをクエリして、読み込みが成功したことを確認できます。

1. 次のステートメントを実行して `table1` のデータをクエリします:

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

1. 次のステートメントを実行して `table2` のデータをクエリします:

   ```SQL
   MySQL [test_db]> SELECT * FROM table2;
   +------+--------+
   | id   | city   |
   +------+--------+
   | 200  | Beijing|
   +------+--------+
   4 rows in set (0.01 sec)
   ```

### 単一テーブルの読み込みジョブの作成

指定されたパスから単一のデータファイルまたはすべてのデータファイルを単一の宛先テーブルに読み込むこともできます。AWS S3 バケット `bucket_s3` に `input` という名前のフォルダが含まれており、そのフォルダには `file1.csv` など複数のデータファイルが含まれていると仮定します。これらのデータファイルは、`table1` の列と同じ数の列から構成されており、それぞれのデータファイルの列は、`table1` の列と順番に 1 対 1 でマッピングできます。

`file1.csv` を `table1` に読み込むには、次のステートメントを実行します:

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

`input` フォルダからすべてのデータファイルを `table1` に読み込むには、次のステートメントを実行します:

```SQL
LOAD LABEL test_db.label_8
(
```
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

前述の例では、`StorageCredentialParams` は、選択した認証方法に応じて異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3) を参照してください。

### ロードジョブの表示

Broker Load を使用すると、`SHOW LOAD` ステートメントまたは `curl` コマンドを使用してロードジョブを表示できます。

#### `SHOW LOAD` の使用

詳細については、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) を参照してください。

#### `curl` の使用

構文は次のとおりです:

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/<database_name>/_load_info?label=<label_name>'
```

> **注意**
>
> パスワードの設定がないアカウントを使用する場合は、`<username>:` のみを入力する必要があります。

たとえば、次のコマンドを実行して、`test_db` データベース内の `label1` というラベルのロードジョブに関する情報を表示できます:

```Bash
curl --location-trusted -u <username>:<password> \
    'http://<fe_host>:<fe_http_port>/api/test_db/_load_info?label=label1'
```

`curl` コマンドは、JSON オブジェクト `jobInfo` としてロードジョブの情報を返します:

```JSON
{"jobInfo":{"dbName":"default_cluster:test_db","tblNames":["table1_simple"],"label":"label1","state":"FINISHED","failMsg":"","trackingUrl":""},"status":"OK","msg":"Success"}%
```

以下の表には、`jobInfo` のパラメータが説明されています。

| **パラメータ** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| dbName        | データがロードされるデータベースの名前           |
| tblNames      | データがロードされるテーブルの名前。             |
| label         | ロードジョブのラベル。                                   |
| state         | ロードジョブの状態。有効な値:<ul><li>`PENDING`: キューに入れられてスケジュール待ちです。</li><li>`QUEUEING`: キューに入れられてスケジュール待ちです。</li><li>`LOADING`: ロードジョブ実行中です。</li><li>`PREPARED`: トランザクションがコミットされました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul> 詳細については、[データロードの概要](../loading/Loading_intro.md) の "非同期ロード" セクションを参照してください。 |
| failMsg       | ロードジョブが失敗した理由。ロードジョブの `state` 値が `PENDING`、`LOADING`、または `FINISHED` である場合、`failMsg` パラメータの戻り値は `NULL` となります。ロードジョブの `state` 値が `CANCELLED` である場合、`failMsg` パラメータの戻り値は `type` と `msg` の 2 つの部分から成ります。<ul><li>`type` の部分は、次のいずれかの値になります:</li><ul><li>`USER_CANCEL`: ロードジョブは手動でキャンセルされました。</li><li>`ETL_SUBMIT_FAIL`: ロードジョブの送信に失敗しました。</li><li>`ETL-QUALITY-UNSATISFIED`: 不適格データの割合が `max-filter-ratio` パラメータの値を超えたため、ロードジョブが失敗しました。</li><li>`LOAD-RUN-FAIL`: ロードジョブが `LOADING` ステージで失敗しました。</li><li>`TIMEOUT`: パフォーマンスが指定されたタイムアウト期間内に終了しませんでした。</li><li>`UNKNOWN`: 不明なエラーによりロードジョブが失敗しました。</li></ul><li>`msg` 部分は、ロード失敗の詳細な原因を提供します。</li></ul> |
| trackingUrl   | ロードジョブで検出された不適格データにアクセスするために使用される URL。`curl` または `wget` コマンドを使用して URL にアクセスし、不適格データを取得できます。不適格データが検出されない場合、`trackingUrl` パラメータの戻り値は `NULL` となります。 |
| status        | ロードジョブの HTTP リクエストの状態。有効な値: `OK` および `Fail`。 |
| msg           | ロードジョブの HTTP リクエストのエラー情報。 |

### ロードジョブのキャンセル

ロードジョブが **CANCELLED** または **FINISHED** ステージにない場合、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) ステートメントを使用してジョブをキャンセルできます。

たとえば、次のステートメントを実行して、`test_db` データベース内の `label1` というラベルのロードジョブをキャンセルできます:

```SQL
CANCEL LOAD
FROM test_db
WHERE LABEL = "label";
```

## ジョブの分割および同時実行

Broker Load ジョブは、複数のタスクに分割して同時に実行できます。ロードジョブ内のタスクはすべて単一トランザクション内で実行される必要があります。それらはすべて成功するか、失敗する必要があります。StarRocks は、`LOAD` ステートメントで `data_desc` を宣言する方法に基づいて、各ロードジョブを分割します:

- 異なるテーブルを指定する複数の `data_desc` パラメータを宣言する場合、各テーブルのデータをロードするタスクが生成されます。

- 同じテーブルの異なるパーティションを指定する複数の `data_desc` パラメータを宣言する場合、各パーティションのデータをロードするタスクが生成されます。

さらに、各タスクは、以下の [ファイル配布配置の設定](../administration/Configuration.md#fe-configuration-items) に基づいて、1 つ以上のインスタンスにさらに分割できます:

- `min_bytes_per_broker_scanner`: 各インスタンスで処理されるデータの最小量。デフォルトの量は 64 MB です。

- `load_parallel_instance_num`: 個々の BE での各ロードジョブで許可される並行インスタンスの数。デフォルトの数は 1 です。
  
個々のタスク内のインスタンスの数を計算するには、次の式を使用できます:

**個々のタスク内のインスタンスの数 = min(個々のタスクでロードするデータ量/`min_bytes_per_broker_scanner`,`load_parallel_instance_num` x BE の数)**

ほとんどの場合、各ロードジョブには1つの `data_desc` のみが宣言され、各ロードジョブは1つのタスクにのみ分割され、タスクは BE の数と同じ数のインスタンスに分割されます。

## 関連する設定項目

[FE 設定項目](../administration/Configuration.md#fe-configuration-items) `max_broker_load_job_concurrency` は、StarRocks クラスタ内で同時に実行できる Broker Load ジョブの最大数を指定します。

StarRocks v2.4 およびそれ以前では、特定の期間内に提出された合計 Broker Load ジョブの数が最大数を超えると、余分なジョブはキューに入れられ、提出された時刻に基づいてスケジュールされます。

StarRocks v2.5 以降では、特定の期間内に提出された合計 Broker Load ジョブの数が最大数を超えると、余分なジョブは優先度に基づいてキューに入れられ、スケジュールされます。ジョブの優先度は、ジョブの作成時に `priority` パラメータを使用して指定できます。詳細は [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#opt_properties) を参照してください。また、**QUEUEING** または **LOADING** 状態の既存のジョブの優先度を変更するには、[ALTER LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md) を使用できます。