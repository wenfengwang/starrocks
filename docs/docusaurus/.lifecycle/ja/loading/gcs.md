---
displayed_sidebar: "Japanese"
---

# GCSからデータを読み込む

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、GCSからデータを読み込むための2つのオプションが用意されています：

1. ブローカープロセスを使用した非同期読み込み
2. `FILES()` テーブル関数を使用した同期読み込み

小規模なデータセットは、`FILES()` テーブル関数を使用して同期的に読み込まれることが多く、大規模なデータセットは、ブローカープロセスを使用して非同期読み込みされることが多いです。これら2つの方法には異なる利点があり、以下で説明します。

<InsertPrivNote />

## 接続詳細を収集する

> **注意**
>
> ここでの例では、サービスアカウントキー認証が使用されています。その他の認証方法については、このページの最下部にリンクがあります。
>
> このガイドでは、StarRocksによってホストされているデータセットが使用されています。このデータセットは、認証済みのGCPユーザーであれば誰でも読み取ることができるため、以下で使用する権限情報を使用してParquetファイルを読み込むことができます。

GCSからデータを読み込むには、以下が必要です：

- GCSバケット
- GCSオブジェクトキー（オブジェクト名）（バケット内の特定のオブジェクトにアクセスする場合）。GCSオブジェクトがサブフォルダに格納されている場合、オブジェクトキーにプレフィックスを含めることができます。完全な構文は**詳細情報**にリンクされています。
- GCSリージョン
- サービスアカウントのアクセスキーとシークレット

## ブローカープロセスの使用

非同期のブローカープロセスは、GCSへの接続、データの取得、およびデータのStarRocksへの格納を処理します。

### ブローカープロセスの利点

- ブローカープロセスはデータ変換、UPSERT、および削除操作を読み込み中にサポートします。
- ブローカープロセスはバックグラウンドで実行され、クライアントが接続を維持する必要はありません。
- 長時間実行するジョブにはブローカープロセスが推奨され、デフォルトのタイムアウトは4時間です。
- ParquetおよびORCファイルフォーマットに加えて、ブローカープロセスはCSVファイルをサポートします。

### データフロー

![ブローカープロセスのワークフロー](../assets/broker_load_how-to-work_en.png)

1. ユーザーが読み込みジョブを作成します。
2. フロントエンド（FE）がクエリプランを作成し、プランをバックエンドノード（BE）に配布します。
3. バックエンド（BE）ノードが元のデータを取得し、データをStarRocksに読み込みます。

### 典型的な例

テーブルを作成し、GCSからParquetファイルを取得する読み込みプロセスを開始し、データの読み込みの進行状況と成功を確認します。

> **注意**
>
> この文書の例では、Parquet形式のサンプルデータセットが使用されています。CSVまたはORCファイルを読み込む場合は、これらの情報がこのページの最下部にリンクされています。

#### テーブルの作成

テーブルのためのデータベースを作成します：

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
> この文書の例では、`replication_num`プロパティが`1`に設定されているため、単一のBEシステムで実行することができます。3つ以上のBEを使用する場合は、DDLの`PROPERTIES`セクションを削除してください。

#### ブローカープロセスの開始

このジョブには、以下の4つの主要なセクションがあります：

- `LABEL`：`LOAD`ジョブの状態をクエリする際に使用される文字列
- `LOAD`宣言：ソースURI、宛先テーブル、およびソースデータ形式
- `BROKER`：ソースの接続詳細
- `PROPERTIES`：このジョブに適用するタイムアウト値とその他のプロパティ

> **注意**
>
> これらの例で使用されているデータセットは、StarRocksアカウントのGCSバケットにホストされています。有効なサービスアカウントのメールアドレス、キー、およびシークレットを使用できます。そのため、以下のコマンドのプレースホルダーに自分の資格情報を代替してください。

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

進行状況を追跡するために`information_schema.loads`テーブルをクエリします。複数の`LOAD`ジョブが実行されている場合は、ジョブに関連付けられた`LABEL`で絞り込むことができます。以下の出力では、`user_behavior`の読み込みジョブに対して2つのエントリがあります。最初のレコードは`CANCELLED`状態を示し、スクロールして最後に行くと`listPath failed`が表示されます。2番目のレコードはAWS IAMアクセスキーとシークレットが有効であることを示し、成功しています。

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

この時点で、データの部分を確認することもできます。

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
```plaintext
171146|458604|   1349561|pv          |2017-11-27 14:03:44|
171146|478802|    541347|pv          |2017-12-02 04:52:39|
```

## `FILES()` テーブル関数の使用

### `FILES()` の利点

`FILES()` は Parquet データの列のデータ型を推測し、StarRocks テーブルのスキーマを生成できます。これにより、S3 から `SELECT` を使用してファイルを直接クエリしたり、Parquet ファイルのスキーマに基づいて StarRocks が自動的にテーブルを作成したりできます。

> **注意**
>
> スキーマ推測は、バージョン 3.1 の新機能であり、Parquet 形式のみをサポートしており、入れ子のタイプはまだサポートされていません。

### 典型的な例

`FILES()` テーブル関数を使用する 3 つの例があります:

- S3 からデータを直接クエリする
- スキーマ推測を使用してテーブルを作成およびロードする
- テーブルを作成してからデータをロードする

#### S3 から直接クエリ

`FILES()` を使用して S3 から直接クエリすることで、テーブルを作成する前にデータセットの内容をプレビューすることが可能です。例:

- データを格納せずにデータセットのプレビューを取得する。
- 最小値と最大値をクエリしてデータ型を決定する。
- NULL をチェックする。

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
> 列名が Parquet ファイルによって提供されていることに注意してください。

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

#### スキーマ推測を使用したテーブルの作成

これは前の例の続きで、以前のクエリを `CREATE TABLE` でラップし、Parquet ファイルのスキーマを用いてテーブル作成を自動化します。Parquet フォーマットには列名と型が含まれているため、`FILES()` テーブル関数を使用してテーブルを作成する際には、列名と型の指定は必要ありません。

> **注意**
>
> スキーマ推測を使用する場合、`CREATE TABLE` の構文ではレプリカ数の設定は許可されていないので、テーブルを作成する前に設定する必要があります。以下の例は、シングルレプリカのシステム用です:
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
> 推測されたスキーマを手動で作成されたスキーマと比較します:
>
> - データ型
> - NULL 可能
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

挿入先のテーブルをカスタマイズしたい場合があります。たとえば:

- 列のデータ型、NULL 可能設定、またはデフォルト値
- キータイプおよび列
- 分散
- 他

> **注意**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法と列の内容を知る必要があります。このドキュメントではテーブル設計については説明していませんが、ページの最後に **more information** というリンクがあります。

この例では、テーブルを作成するために Parquet ファイルのデータに対する知識を使用しています。

- S3 でのファイルのクエリによって `Timestamp` 列が `datetime` データ型に一致するデータを含むことがわかるため、次の DDL で列の型が指定されています。
- S3 でのデータのクエリによって、データセットに NULL 値が存在しないことが分かるため、DDL では列を NULL 不可としていません。
- 期待されるクエリタイプの知識に基づいて、ソートキーとバケット化される列が `UserID` に設定されます (`UserID` をソートキーとして使用するか、または `UserID` の代わりに `ItemID` を使用するかは、ご利用のユースケースによって異なる場合があります)。

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

テーブルを作成した後、`INSERT INTO` … `SELECT FROM FILES()` でテーブルにデータをロードできます:

```SQL
INSERT INTO user_behavior_declared
  SELECT * FROM FILES(
    "path" = "gs://starrocks-samples/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
```yaml
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
);
```

## さらなる情報

- このドキュメントはサービスアカウントキー認証のみをカバーしています。その他のオプションについては[Google Cloud Storageリソースへの認証](../integrations/authenticate_to_gcs.md)を参照してください。
- 同期および非同期データロードの詳細については、[データローディングの概要](../loading/Loading_intro.md)ドキュメントをご覧ください。
- ローディング中にデータ変換をサポートするブローカーロードについては、[ローディング時のデータ変換](../loading/Etl_in_loading.md)と[ローディングを通じたデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。
- [テーブルデザイン](../table_design/StarRocks_table_design.md)についてもっと詳しく学んでください。
- 上記の例よりもブローカーロードには多くの構成オプションや使用オプションがあります。詳細については[ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)をご覧ください。
```