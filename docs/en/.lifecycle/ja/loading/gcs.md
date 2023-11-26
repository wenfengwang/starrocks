---
displayed_sidebar: "Japanese"
---

# GCSからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、GCSからデータをロードするための2つのオプションが提供されています。

1. ブローカーロードを使用した非同期ロード
2. `FILES()`テーブル関数を使用した同期ロード

小規模なデータセットは、`FILES()`テーブル関数を使用して同期的にロードされ、大規模なデータセットはブローカーロードを使用して非同期にロードされることがよくあります。これらの2つの方法には異なる利点があり、以下で説明します。

<InsertPrivNote />

## 接続の詳細を取得する

> **注意**
>
> このガイドでは、サービスアカウントキー認証を使用しています。他の認証方法も利用可能であり、このページの最下部にリンクがあります。
>
> このガイドでは、StarRocksによってホストされているデータセットを使用しています。このデータセットは、認証されたGCPユーザーであれば誰でも読み取ることができるため、以下で使用されるParquetファイルを読み取るために資格情報を使用することができます。

GCSからデータをロードするには、以下の情報が必要です。

- GCSバケット
- GCSオブジェクトキー（オブジェクト名）（バケット内の特定のオブジェクトにアクセスする場合）。GCSオブジェクトがサブフォルダに格納されている場合、オブジェクトキーには接頭辞を含めることができます。完全な構文は**詳細情報**にリンクされています。
- GCSリージョン
- サービスアカウントのアクセスキーとシークレット

## ブローカーロードの使用

非同期のブローカーロードプロセスは、GCSへの接続、データの取得、およびデータのStarRocksへの保存を処理します。

### ブローカーロードの利点

- ブローカーロードは、ロード中にデータの変換、UPSERT、およびDELETE操作をサポートしています。
- ブローカーロードはバックグラウンドで実行され、ジョブが続行するためにクライアントが接続されている必要はありません。
- ブローカーロードは長時間実行されるジョブに適しており、デフォルトのタイムアウトは4時間です。
- ParquetおよびORCファイル形式に加えて、ブローカーロードはCSVファイルもサポートしています。

### データフロー

![ブローカーロードのワークフロー](../assets/broker_load_how-to-work_en.png)

1. ユーザーがロードジョブを作成します。
2. フロントエンド（FE）がクエリプランを作成し、プランをバックエンドノード（BE）に配布します。
3. バックエンド（BE）ノードがソースからデータを取得し、データをStarRocksにロードします。

### 典型的な例

テーブルを作成し、GCSからParquetファイルを取得するロードプロセスを開始し、データのロードの進行状況と成功を確認します。

> **注意**
>
> このドキュメントの例では、Parquet形式のサンプルデータセットを使用しています。CSVファイルまたはORCファイルをロードする場合は、このページの最下部にリンクされている情報を参照してください。

#### テーブルの作成

テーブル用のデータベースを作成します。

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

テーブルを作成します。このスキーマは、StarRocksアカウントでホストされているGCSバケット内のサンプルデータセットと一致します。

```SQL
DROP TABLE IF EXISTS user_behavior;

CREATE TABLE `user_behavior` (
    `UserID` int(11),
    `ItemID` int(11),
    `CategoryID` int(11),
    `BehaviorType` varchar(65533),
    `Timestamp` datetime
) ENGINE=OLAP 
DUPLICATE KEY(`UserID`)
DISTRIBUTED BY HASH(`UserID`)
PROPERTIES (
    "replication_num" = "1"
);
```

> **注意**
>
> このドキュメントの例では、プロパティ`replication_num`が`1`に設定されているため、シンプルな単一のBEシステムで実行できます。3つ以上のBEを使用している場合は、DDLの`PROPERTIES`セクションを削除してください。

#### ブローカーロードの開始

このジョブには、4つの主要なセクションがあります。

- `LABEL`：`LOAD`ジョブの状態をクエリする際に使用される文字列です。
- `LOAD`宣言：ソースURI、宛先テーブル、およびソースデータ形式です。
- `BROKER`：ソースの接続詳細です。
- `PROPERTIES`：このジョブに適用するタイムアウト値およびその他のプロパティ。

> **注意**
>
> これらの例では、GCSバケット内のサンプルデータセットを使用しています。GCSバケット内の任意の有効なサービスアカウントのメール、キー、およびシークレットを使用できます。なお、このオブジェクトは、GCPの認証済みユーザーであれば誰でも読み取ることができます。以下のコマンドのプレースホルダーに自分の資格情報を置き換えてください。

```SQL
LOAD LABEL user_behavior
(
    DATA INFILE("gs://starrocks-samples/user_behavior_ten_million_rows.parquet")
    INTO TABLE user_behavior
    FORMAT AS "parquet"
 )
 WITH BROKER
 (
 
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
 )
PROPERTIES
(
    "timeout" = "72000"
);
```

#### 進行状況の確認

進行状況を追跡するために`information_schema.loads`テーブルをクエリします。複数の`LOAD`ジョブが実行されている場合は、ジョブに関連付けられた`LABEL`でフィルタリングすることができます。以下の出力では、`user_behavior`のロードジョブに2つのエントリがあります。最初のレコードは`CANCELLED`の状態を示しており、出力の最後にスクロールすると`listPath failed`と表示されます。2番目のレコードは、有効なAWS IAMアクセスキーとシークレットを持つ成功を示しています。

```SQL
SELECT * FROM information_schema.loads;
```

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'user_behavior';
```

```plaintext
JOB_ID|LABEL                                      |DATABASE_NAME|STATE    |PROGRESS           |TYPE  |PRIORITY|SCAN_ROWS|FILTERED_ROWS|UNSELECTED_ROWS|SINK_ROWS|ETL_INFO|TASK_INFO                                           |CREATE_TIME        |ETL_START_TIME     |ETL_FINISH_TIME    |LOAD_START_TIME    |LOAD_FINISH_TIME   |JOB_DETAILS                                                                                                                                                                                                                                                    |ERROR_MSG                             |TRACKING_URL|TRACKING_SQL|REJECTED_RECORD_PATH|
------+-------------------------------------------+-------------+---------+-------------------+------+--------+---------+-------------+---------------+---------+--------+----------------------------------------------------+-------------------+-------------------+-------------------+-------------------+-------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------------------------------------+------------+------------+--------------------+
 10121|user_behavior                              |project      |CANCELLED|ETL:N/A; LOAD:N/A  |BROKER|NORMAL  |        0|            0|              0|        0|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:59:30|                   |                   |                   |2023-08-10 14:59:34|{"All backends":{},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":0,"InternalTableLoadRows":0,"ScanBytes":0,"ScanRows":0,"TaskNumber":0,"Unfinished backends":{}}                                                                                        |type:ETL_RUN_FAIL; msg:listPath failed|            |            |                    |
 10106|user_behavior                              |project      |FINISHED |ETL:100%; LOAD:100%|BROKER|NORMAL  | 86953525|            0|              0| 86953525|        |resource:N/A; timeout(s):72000; max_filter_ratio:0.0|2023-08-10 14:50:15|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:50:19|2023-08-10 14:55:10|{"All backends":{"a5fe5e1d-d7d0-4826-ba99-c7348f9a5f2f":[10004]},"FileNumber":1,"FileSize":1225637388,"InternalTableLoadBytes":2710603082,"InternalTableLoadRows":86953525,"ScanBytes":1225637388,"ScanRows":86953525,"TaskNumber":1,"Unfinished backends":{"a5|                                      |            |            |                    |
```

この時点でデータの一部を確認することもできます。

```SQL
SELECT * from user_behavior LIMIT 10;
```

```plaintext
UserID|ItemID|CategoryID|BehaviorType|Timestamp          |
------+------+----------+------------+-------------------+
171146| 68873|   3002561|pv          |2017-11-30 07:11:14|
171146|146539|   4672807|pv          |2017-11-27 09:51:41|
171146|146539|   4672807|pv          |2017-11-27 14:08:33|
171146|214198|   1320293|pv          |2017-11-25 22:38:27|
171146|260659|   4756105|pv          |2017-11-30 05:11:25|
171146|267617|   4565874|pv          |2017-11-27 14:01:25|
171146|329115|   2858794|pv          |2017-12-01 02:10:51|
171146|458604|   1349561|pv          |2017-11-25 22:49:39|
171146|458604|   1349561|pv          |2017-11-27 14:03:44|
171146|478802|    541347|pv          |2017-12-02 04:52:39|
```

## `FILES()`テーブル関数の使用

### `FILES()`の利点

`FILES()`は、Parquetデータの列のデータ型を推論し、StarRocksテーブルのスキーマを生成することができます。これにより、`SELECT`を使用してS3から直接ファイルをクエリしたり、Parquetファイルのスキーマに基づいてStarRocksが自動的にテーブルを作成したりすることができます。

> **注意**
>
> スキーマ推論はバージョン3.1でのみ利用可能であり、Parquet形式のみをサポートしており、ネストされた型はまだサポートされていません。

### 典型的な例

`FILES()`テーブル関数を使用した3つの例があります。

- S3からデータを直接クエリする
- スキーマ推論を使用してテーブルを作成およびロードする
- 手動でテーブルを作成し、データをロードする

#### S3から直接クエリする

`FILES()`を使用してS3から直接クエリすることで、テーブルを作成する前にデータセットの内容をプレビューすることができます。例えば：

- データを保存せずにデータセットのプレビューを取得します。
- 最小値と最大値をクエリして使用するデータ型を決定します。
- null値をチェックします。

```sql
SELECT * FROM FILES(
    "path" = "gs://starrocks-samples/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
) LIMIT 10;
```

> **注意**
>
> カラム名はParquetファイルによって提供されることに注意してください。

```plaintext
UserID|ItemID |CategoryID|BehaviorType|Timestamp          |
------+-------+----------+------------+-------------------+
     1|2576651|    149192|pv          |2017-11-25 01:21:25|
     1|3830808|   4181361|pv          |2017-11-25 07:04:53|
     1|4365585|   2520377|pv          |2017-11-25 07:49:06|
     1|4606018|   2735466|pv          |2017-11-25 13:28:01|
     1| 230380|    411153|pv          |2017-11-25 21:22:22|
     1|3827899|   2920476|pv          |2017-11-26 16:24:33|
     1|3745169|   2891509|pv          |2017-11-26 19:44:31|
     1|1531036|   2920476|pv          |2017-11-26 22:02:12|
     1|2266567|   4145813|pv          |2017-11-27 00:11:11|
     1|2951368|   1080785|pv          |2017-11-27 02:47:08|
```

#### スキーマ推論を使用したテーブルの作成

これは前の例の続きです。前のクエリを`CREATE TABLE`でラップして、スキーマ推論を使用してテーブルの作成を自動化します。列名と型は、Parquetファイルのスキーマが含まれているため、テーブルを作成するために必要ありません。

> **注意**
>
> スキーマ推論を使用する場合の`CREATE TABLE`の構文では、レプリカ数を設定することはできないため、テーブルを作成する前に設定してください。以下の例は、シングルレプリカのシステムの場合のものです：
>
> `ADMIN SET FRONTEND CONFIG ('default_replication_num' ="1");`

```sql
CREATE DATABASE IF NOT EXISTS project;
USE project;

CREATE TABLE `user_behavior_inferred` AS
SELECT * FROM FILES(
    "path" = "gs://starrocks-samples/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
);
```

```SQL
DESCRIBE user_behavior_inferred;
```

```plaintext
Field       |Type            |Null|Key  |Default|Extra|
------------+----------------+----+-----+-------+-----+
UserID      |bigint          |YES |true |       |     |
ItemID      |bigint          |YES |true |       |     |
CategoryID  |bigint          |YES |true |       |     |
BehaviorType|varchar(1048576)|YES |false|       |     |
Timestamp   |varchar(1048576)|YES |false|       |     |
```

> **注意**
>
> 推論されたスキーマを手動で作成されたスキーマと比較してください：
>
> - データ型
> - nullable
> - キーフィールド

```SQL
SELECT * from user_behavior_inferred LIMIT 10;
```

```plaintext
UserID|ItemID|CategoryID|BehaviorType|Timestamp          |
------+------+----------+------------+-------------------+
171146| 68873|   3002561|pv          |2017-11-30 07:11:14|
171146|146539|   4672807|pv          |2017-11-27 09:51:41|
171146|146539|   4672807|pv          |2017-11-27 14:08:33|
171146|214198|   1320293|pv          |2017-11-25 22:38:27|
171146|260659|   4756105|pv          |2017-11-30 05:11:25|
171146|267617|   4565874|pv          |2017-11-27 14:01:25|
171146|329115|   2858794|pv          |2017-12-01 02:10:51|
171146|458604|   1349561|pv          |2017-11-25 22:49:39|
171146|458604|   1349561|pv          |2017-11-27 14:03:44|
171146|478802|    541347|pv          |2017-12-02 04:52:39|
```

#### 既存のテーブルにロードする

挿入先のテーブルをカスタマイズする場合があります。たとえば：

- カラムのデータ型、nullableの設定、デフォルト値をカスタマイズする
- キータイプとカラムをカスタマイズする
- ディストリビューションをカスタマイズする
- など

> **注意**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法と列の内容を知る必要があります。このドキュメントではテーブル設計について説明しておらず、このページの最下部にリンクがあります。

この例では、クエリの方法とParquetファイルのデータに関する知識に基づいてテーブルを作成します。Parquetファイルのデータに関する知識は、S3でファイルをクエリすることで得ることができます。

- S3でのファイルのクエリによって、`Timestamp`列に`datetime`データ型に一致するデータが含まれていることがわかるため、次のDDLで列の型が指定されています。
- S3でデータをクエリすることで、データセットにnull値がないことがわかるため、DDLでは列をnullableとして設定していません。
- 予想されるクエリタイプに基づいて、ソートキーとバケット化カラムを列`UserID`に設定しています（このデータの場合、ソートキーには`UserID`の他に`ItemID`を使用するか、または`UserID`の代わりに`ItemID`を使用するか、または両方を使用するかは、ユースケースにより異なるかもしれません）：

```SQL
CREATE TABLE `user_behavior_declared` (
    `UserID` int(11),
    `ItemID` int(11),
    `CategoryID` int(11),
    `BehaviorType` varchar(65533),
    `Timestamp` datetime
) ENGINE=OLAP 
DUPLICATE KEY(`UserID`)
DISTRIBUTED BY HASH(`UserID`)
PROPERTIES (
    "replication_num" = "1"
);
```

テーブルを作成した後、`INSERT INTO` … `SELECT FROM FILES()`でテーブルにデータをロードできます。

```SQL
INSERT INTO user_behavior_declared
  SELECT * FROM FILES(
    "path" = "gs://starrocks-samples/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
);
```

## 詳細情報

- このドキュメントでは、サービスアカウントキー認証のみをカバーしています。その他のオプションについては、[GCSリソースへの認証](../integrations/authenticate_to_gcs.md)を参照してください。
- 同期および非同期のデータロードの詳細については、[データロードの概要](../loading/Loading_intro.md)ドキュメントを参照してください。
- ブローカーロードがロード中にデータ変換をサポートする方法については、[ロード時のデータ変換](../loading/Etl_in_loading.md)と[ロードを介したデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。
- [テーブル設計](../table_design/StarRocks_table_design.md)について詳しくは、リンクを参照してください。
- ブローカーロードには、上記の例に記載されているものよりも多くの設定と使用オプションがあります。詳細については、[ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。
