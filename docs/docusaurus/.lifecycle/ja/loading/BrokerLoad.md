---
displayed_sidebar: "Japanese"
---

# HDFSまたはクラウドストレージからデータをロード

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、HDFSまたはクラウドストレージから大量のデータをStarRocksにロードするのを助けるために、MySQLベースのBroker Loadローディングメソッドを提供しています。

ブローカーロードは非同期ローディングモードで実行されます。ロードジョブを送信すると、StarRocksは非同期でジョブを実行します。ロードジョブの結果を確認するには、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) ステートメントまたは`curl`コマンドを使用する必要があります。

ブローカーロードは、単一テーブルのロードおよび複数テーブルのロードをサポートしています。1つのブローカーロードジョブを実行することで、1つまたは複数のデータファイルを1つまたは複数の宛先テーブルにロードできます。ブローカーロードは、実行される複数のデータファイルのロードジョブのトランザクション的な原子性を保証します。原子性とは、1つのロードジョブで複数のデータファイルのロードがすべて成功するか、失敗するかのいずれかであることを意味します。1部のデータファイルのロードが成功し、他のファイルのロードが失敗することはありません。

ブローカーロードは、データのローディング時のデータ変換をサポートし、ローディング時のUPSERTおよびDELETE操作によるデータの変更をサポートしています。詳細は[ローディング時のデータ変換](../loading/Etl_in_loading.md)および[ローディングを介したデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

<InsertPrivNote />

## バックグラウンド情報

v2.4以前では、StarRocksはブローカーロードジョブを実行する際にStarRocksクラスターと外部ストレージシステムの間に接続を設定するためにブローカーに依存していました。そのため、ロードステートメントで使用するブローカーを指定するには、「WITH BROKER "<broker_name>"」を入力する必要がありました。これを「ブローカーベースのローディング」と呼びます。ブローカーはファイルシステムインターフェースに統合された独立したステートレスサービスです。ブローカーを使用することで、StarRocksは外部ストレージシステムに保存されているデータファイルにアクセスして読み取り、これらのデータファイルのデータを事前処理してロードするために独自のコンピューティングリソースを使用できます。

v2.5以降では、StarRocksはブローカーロードジョブを実行する際にStarRocksクラスターと外部ストレージシステムの間に接続を設定するためにブローカーに依存しなくなりました。そのため、ロードステートメントでブローカーを指定する必要はなくなりましたが、引き続き`WITH BROKER`キーワードを保持する必要があります。これを「ブローカーフリーローディング」と呼びます。

データがHDFSに格納されている場合、ブローカーフリーローディングが機能しないことがあります。これは、データが複数のHDFSクラスターにまたがって配置されている場合や、複数のKerberosユーザーを構成している場合に発生する可能性があります。これらの状況では、ブローカーベースのローディングに戻ることができます。これを成功させるには、少なくとも1つの独立したブローカーグループがデプロイされていることを確認してください。これらの状況での認証構成とHA構成の指定方法については、[HDFS](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。

> **注記**
>
> [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) ステートメントを使用して、StarRocksクラスターに展開されているブローカーを確認できます。ブローカーが展開されていない場合は、[ブローカーを展開](../deployment/deploy_broker.md)するための指示に従ってブローカーを展開できます。

## サポートされているデータファイルフォーマット

ブローカーロードは、以下のデータファイルフォーマットをサポートしています。

- CSV

- Parquet

- ORC

> **注記**
>
> CSVデータの場合、以下の点に注意してください:
>
> - コンマ(,)、タブ、またはパイプ(|)など、長さが50バイトを超えないUTF-8文字列をテキストデリミタとして使用できます。
> - Null値は`\N`を使用して示します。たとえば、データファイルは3つの列から構成され、そのデータファイルのレコードは最初の列と三番目の列にデータを保持しますが、二番目の列にデータはありません。このような状況では、二番目の列にNull値を示すために`\N`を使用する必要があります。これは、レコードは`a,\N,b`としてコンパイルする必要があることを意味します。`a,,b`は、レコードの二番目の列が空の文字列を保持していることを示します。

## サポートされているストレージシステム

ブローカーロードは、以下のストレージシステムをサポートしています:

- HDFS

- AWS S3

- Google GCS

- MinIOなどの他のS3互換ストレージシステム

- Microsoft Azure Storage

## 動作方法

FEにロードジョブを送信すると、FEはクエリプランを生成し、利用可能なBEの数とロードしたいデータファイルのサイズに基づいてクエリプランを部分に分割し、それぞれのクエリプランの部分を利用可能なBEに割り当てます。ロード中、各関与するBEは、HDFSまたはクラウドストレージシステムからデータファイルのデータを取得し、データを事前処理してからStarRocksクラスターにデータをロードします。すべてのBEがクエリプランの部分を完了した後、FEはロードジョブが成功したかどうかを判断します。

以下の図は、ブローカーロードジョブのワークフローを示しています。

![ブローカーロードのワークフロー](../assets/broker_load_how-to-work_en.png)

## 基本操作

### 複数のテーブルのロードジョブを作成する

このトピックでは、CSVを例にして、複数のデータファイルを複数のテーブルにロードする方法について説明します。他のファイルフォーマットでデータをロードする方法や、ブローカーロードの構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

StarRocksでは、一部のリテラルがSQL言語によって予約されたキーワードとして使用されています。これらのキーワードはSQLステートメントで直接使用しないでください。SQLステートメントでそのようなキーワードを使用する場合は、バックティック（`）で囲んでください。[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データ例

1. ローカルファイルシステムにCSVファイルを作成します。

   a. `file1.csv`という名前のCSVファイルを作成します。このファイルは3つの列から構成されており、順にユーザーID、ユーザー名、およびユーザースコアを表します。

      ```Plain
      1,Lily,23
      2,Rose,23
      3,Alice,24
      4,Julia,25
      ```

   b. `file2.csv`という名前のCSVファイルを作成します。このファイルは2つの列から構成されており、順に都市IDと都市名を表します。

      ```Plain
      200,'Beijing'
      ```

2. StarRocksデータベース`test_db`にPrimary Keyテーブルを作成します。

   > **注記**
   >
   > v2.5.7以降、StarRocksでは、テーブルを作成したりパーティションを追加したりする際にバケツ数（BUCKETS）を自動的に設定することができます。バケツ数を手動で設定する必要はもはやありません。詳細については、[バケツ数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

   a. `table1`という名前のPrimary Keyテーブルを作成します。このテーブルは、`id`、`name`、および`score`の3つの列から構成されています。`id`は主キーです。

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

   b. `table2`という名前のPrimary Keyテーブルを作成します。このテーブルは、`id`と`city`の2つの列から構成されており、`id`が主キーです。

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

3. `file1.csv`と`file2.csv`をHDFSクラスターの`/user/starrocks/`パス、AWS S3バケット`bucket_s3`の`input`フォルダ、Google GCSバケット`bucket_gcs`の`input`フォルダ、MinIOバケット`bucket_minio`の`input`フォルダ、およびAzure Storageの指定されたパスにアップロードします。

#### HDFSからデータをロード

次のステートメントを実行して、HDFSクラスターの`/user/starrocks`パスから`file1.csv`および`file2.csv`をそれぞれ`table1`および`table2`にロードします:

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

前述の例では、「StorageCredentialParams」は、選択した認証メソッドに応じて異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。

#### AWS S3 からデータをロード

次のステートメントを実行して、`file1.csv`および`file2.csv`をAWS S3バケット`bucket_s3`の`input`フォルダからそれぞれ`table1`および`table2`にロードします。

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
> ブローカーロードは、S3Aプロトコルに基づいてAWS S3へのアクセスのみをサポートしています。したがって、AWS S3からデータをロードする場合は、ファイルパスとして渡すS3 URI内の`s3://`を`s3a://`に置換する必要があります。

前述の例では、「StorageCredentialParams」は、選択した認証メソッドに応じて異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#aws-s3)を参照してください。

v3.1から、StarRocksはINSERTコマンドとTABLEキーワードを使用して、Parquet形式またはORC形式のファイルのデータをAWS S3から直接ロードすることをサポートしています。これにより、最初に外部テーブルを作成する手間が省かれます。詳細については、[INSERTを使用したデータのロード > TABLEキーワードを使用して外部ソースから直接ファイルを挿入する](../loading/InsertInto.md#insert-data-directly-from-files-in-an-external-source-using-table-keyword)を参照してください。

#### Google GCS からデータをロード

次のステートメントを実行して、`file1.csv`および`file2.csv`をGoogle GCSバケット`bucket_gcs`の`input`フォルダからそれぞれ`table1`および`table2`にロードします。

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
> ブローカーロードは、gsプロトコルに基づいたGoogle GCSへのアクセスのみをサポートしています。したがって、Google GCSからデータをロードする場合は、ファイルパスとして渡すGCS URI内に`gs://`を含める必要があります。

前述の例では、「StorageCredentialParams」は、選択した認証メソッドに応じて異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#google-gcs)を参照してください。

#### 他のS3互換ストレージシステムからデータをロード

MinIOを例に取ると、次のステートメントを実行して、MinIOバケット`bucket_minio`の`input`フォルダから`file1.csv`および`file2.csv`をそれぞれ`table1`および`table2`にロードします。

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

前述の例では、「StorageCredentialParams」は、選択した認証メソッドに応じて異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#other-s3-compatible-storage-system)を参照してください。

#### Microsoft Azure Storage からデータをロード

指定したパスからAzure Storageの`file1.csv`および`file2.csv`をロードするには、次のステートメントを実行します。

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

> **注意点**
>
> Azure Storageからデータをロードする場合は、アクセスプロトコルと使用する特定のストレージサービスに基づいてどのプレフィックスを使用するかを決定する必要があります。前述の例ではBlob Storageを使用しています。
>
> - Blob Storageからデータをロードする場合、ストレージアカウントへのアクセスがHTTP経由のみを許可する場合は、ファイルパス内に`wasb://`を使用します。例：`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
> - Blob Storageからデータをロードする場合、ストレージアカウントへのアクセスがHTTPS経由のみを許可する場合は、ファイルパス内に`wasbs://`を使用します。例：`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
> - Data Lake Storage Gen1からデータをロードする場合、ファイルパス内に`adl://`を使用します。例：`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
> - Data Lake Storage Gen2からデータをロードする場合、アクセスプロトコルに基づいてファイルパス内に`abfs://`または `abfss://` を使用します。
>   - Data Lake Storage Gen2がHTTP経由のみを許可する場合は、ファイルパス内に`abfs://`を使用します。例：`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。
>   - Data Lake Storage Gen2がHTTPS経由のみを許可する場合は、ファイルパス内に`abfss://`を使用します。例：`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

前述の例では、「StorageCredentialParams」は、選択した認証メソッドに応じて異なる認証パラメータのグループを表します。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#microsoft-azure-storage)を参照してください。

#### データのクエリ

HDFSクラスタ、AWS S3バケット、またはGoogle GCSバケットからのデータのロードが完了したら、SELECTステートメントを使用して、StarRocksテーブルのデータをクエリして、ロードが成功したことを検証できます。

1. 次のステートメントを実行して、`table1`のデータをクエリします:

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

1. 次のステートメントを実行して、`table2`のデータをクエリします:

   ```SQL
   MySQL [test_db]> SELECT * FROM table2;
   +------+--------+
   | id   | city   |
   +------+--------+
   | 200  | Beijing|
   +------+--------+
   4 rows in set (0.01 sec)
   ```

### 単一テーブルのデータロードジョブの作成

指定したパスから単一のデータファイルまたはすべてのデータファイルを単一の宛先テーブルにロードすることもできます。AWS S3バケット`bucket_s3`に`input`という名前のフォルダが含まれているとします。`input`フォルダには複数のデータファイルが含まれており、そのうちの1つは`file1.csv`という名前です。これらのデータファイルは`table1`と同じ列数を持ち、それぞれのデータファイルの各列は順番に`table1`の列に1対1でマッピングできます。

`file1.csv`を`table1`にロードするには、次のステートメントを実行します:

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

`input`フォルダからすべてのデータファイルを`table1`にロードするには、次のステートメントを実行します:

```SQL
LOAD LABEL test_db.label_8
(
```yaml
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