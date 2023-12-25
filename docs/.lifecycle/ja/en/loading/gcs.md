---
displayed_sidebar: English
---

# GCS からデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks は、GCS からデータをロードするための 2 つのオプションを提供します:

1. Broker Load を使用した非同期ロード
2. `FILES()` テーブル関数を使用した同期ロード

小規模なデータセットはしばしば `FILES()` テーブル関数を使用して同期的にロードされ、大規模なデータセットは Broker Load を使用して非同期にロードされます。これら 2 つの方法はそれぞれ異なる利点があり、以下で説明します。

<InsertPrivNote />

## 接続情報の収集

> **注記**
>
> 例ではサービスアカウントキー認証を使用しています。他の認証方法については、このページの下部にリンクがあります。
>
> このガイドでは、StarRocks によってホストされるデータセットを使用します。このデータセットは任意の認証済み GCP ユーザーが読み取り可能ですので、
以下で使用する Parquet ファイルを読み取るためにご自身の認証情報を使用できます。

GCS からデータをロードするには以下が必要です:

- GCS バケット
- GCS オブジェクトキー（オブジェクト名）は、バケット内の特定のオブジェクトにアクセスする場合に必要です。GCS オブジェクトがサブフォルダーに保存されている場合、オブジェクトキーにはプレフィックスを含めることができます。完全な構文は **詳細情報** にリンクされています。
- GCS リージョン
- サービスアカウント アクセスキーとシークレット

## Broker Load の使用

非同期の Broker Load プロセスは、GCS への接続、データの取得、および StarRocks へのデータの格納を処理します。

### Broker Load の利点

- Broker Load は、ロード中のデータ変換、UPSERT、DELETE 操作をサポートします。
- Broker Load はバックグラウンドで実行され、クライアントはジョブが続行されるために接続されたままでいる必要はありません。
- 長時間実行されるジョブには Broker Load が推奨され、デフォルトのタイムアウトは 4 時間です。
- Parquet と ORC ファイル形式に加えて、Broker Load は CSV ファイルもサポートします。

### データフロー

![Broker Load のワークフロー](../assets/broker_load_how-to-work_en.png)

1. ユーザーがロードジョブを作成します。
2. フロントエンド（FE）がクエリプランを作成し、バックエンドノード（BE）にプランを配布します。
3. バックエンドノード（BE）がソースからデータを取得し、StarRocks にデータをロードします。

### 典型的な例

テーブルを作成し、GCS から Parquet ファイルを取得するロードプロセスを開始し、データロードの進捗と成功を確認します。

> **注記**
>
> これらの例では Parquet 形式のサンプルデータセットを使用しています。CSV または ORC ファイルをロードしたい場合は、このページの下部にリンクがあります。

#### テーブルの作成

テーブル用のデータベースを作成します:

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

テーブルを作成します。このスキーマは、StarRocks アカウントでホストされている GCS バケット内のサンプルデータセットに一致します。

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

> **注記**
>
> このドキュメントの例では、プロパティ `replication_num` を `1` に設定しています。これは単純な単一 BE システムで実行するためです。3 つ以上の BE を使用している場合は、DDL の `PROPERTIES` セクションを削除してください。

#### Broker Load を開始する

このジョブには 4 つの主要なセクションがあります:

- `LABEL`: `LOAD` ジョブの状態を照会する際に使用される文字列。
- `LOAD` 宣言: ソース URI、宛先テーブル、およびソースデータ形式。
- `BROKER`: ソースへの接続詳細。
- `PROPERTIES`: タイムアウト値とこのジョブに適用するその他のプロパティ。

> **注記**
>
> これらの例で使用されるデータセットは、StarRocks アカウントの GCS バケットでホストされています。オブジェクトは GCP 認証ユーザーなら誰でも読み取り可能ですので、有効なサービスアカウントのメールアドレス、キー、シークレットを使用できます。以下のコマンドのプレースホルダーをご自身の認証情報に置き換えてください。

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

#### 進捗の確認

`information_schema.loads` テーブルをクエリして進捗を追跡します。複数の `LOAD` ジョブが実行されている場合は、関連する `LABEL` でフィルタリングできます。以下の出力には `user_behavior` のロードジョブに関する 2 つのエントリがあります。最初のレコードは `CANCELLED` 状態を示し、出力の最後には `listPath failed` と表示されます。2 番目のレコードは、有効な AWS IAM アクセスキーとシークレットで成功したことを示しています。

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

## `FILES()` テーブル関数の使用

### `FILES()` の利点

`FILES()` はParquetデータの列のデータ型を推測し、StarRocksテーブルのスキーマを生成することができます。これにより、S3から直接ファイルを`SELECT`でクエリする能力が提供されるほか、Parquetファイルスキーマに基づいてStarRocksが自動的にテーブルを作成することも可能になります。

> **注記**
>
> スキーマ推論はバージョン3.1の新機能であり、Parquet形式のみに提供されており、ネストされた型はまだサポートされていません。

### 典型的な例

`FILES()` テーブル関数を使用する3つの例を紹介します：

- S3から直接データをクエリする
- スキーマ推論を使用してテーブルを作成し、ロードする
- 手動でテーブルを作成し、その後データをロードする

#### S3から直接クエリする

`FILES()` を使用してS3から直接クエリすると、テーブルを作成する前にデータセットの内容をよく確認できます。例えば：

- データを保存せずにデータセットのプレビューを取得する。
- 最小値と最大値をクエリして、使用するデータ型を決定する。
- NULL値をチェックする。

```sql
SELECT * FROM FILES(
    "path" = "gs://starrocks-samples/user_behavior_ten_million_rows.parquet",
    "format" = "parquet",
    "gcp.gcs.service_account_email" = "sampledatareader@xxxxx-xxxxxx-000000.iam.gserviceaccount.com",
    "gcp.gcs.service_account_private_key_id" = "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "gcp.gcs.service_account_private_key" = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
) LIMIT 10;
```

> **注記**
>
> 列名はParquetファイルによって提供されていることに注意してください。

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

これは前の例の続きで、`CREATE TABLE` を使用してスキーマ推論を利用し、テーブルの作成を自動化します。Parquetファイルを使用する場合、`FILES()` テーブル関数を使用すると、Parquet形式には列名と型が含まれているため、StarRocksがスキーマを推測することができ、テーブルを作成する際に列名と型を指定する必要はありません。

> **注記**
>
> スキーマ推論を使用する`CREATE TABLE`の構文では、レプリカの数を設定することはできませんので、テーブルを作成する前に設定してください。以下の例は、シングルレプリカシステム用です：
>
> `ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");`

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

> **注記**
>
> 推論されたスキーマと手作業で作成されたスキーマを比較してください：
>
> - データ型
> - NULL許容
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

挿入するテーブルをカスタマイズしたい場合があります。例えば：

- 列のデータ型、NULL許容設定、またはデフォルト値
- キーの種類と列
- 分散
- など

> **注記**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法と列の内容に関する知識が必要です。このドキュメントではテーブル設計については詳しくは触れていませんが、**詳細情報**にはページの最後にリンクがあります。

この例では、テーブルがどのようにクエリされるか、およびParquetファイル内のデータに関する知識に基づいてテーブルを作成しています。Parquetファイル内のデータに関する知識は、S3でファイルを直接クエリすることで得られます。

- S3でのファイルクエリによると、`Timestamp` 列には`datetime` データ型に一致するデータが含まれているため、以下のDDLでは列の型を指定しています。
- S3でデータをクエリすることで、データセットにNULL値がないことがわかるため、DDLではどの列もNULL許容として設定されていません。
- 想定されるクエリタイプに関する知識に基づき、ソートキーとバケット列は`UserID` 列に設定されています（このデータに対するユースケースは異なるかもしれません。たとえば、ソートキーとして`ItemID` を`UserID` に加えて、または代わりに使用することを決定するかもしれません）。

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

テーブルを作成した後、`INSERT INTO` … `SELECT FROM FILES()`を使用してロードできます：

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

- このドキュメントではサービスアカウントキー認証のみをカバーしています。他のオプションについては、[GCSリソースへの認証](../integrations/authenticate_to_gcs.md)をご覧ください。
- 同期および非同期のデータロードの詳細については、[データロードの概要](../loading/Loading_intro.md)のドキュメントを参照してください。
- ロード中にデータ変換をサポートするブローカーロードについては、[ロード時のデータ変換](../loading/Etl_in_loading.md)と[ロードを通じてデータを変更する](../loading/Load_to_Primary_Key_tables.md)をご覧ください。
- [StarRocksテーブル設計](../table_design/StarRocks_table_design.md)についてもっと学びましょう。
- Broker Loadは上記の例よりも多くの設定オプションと使用オプションを提供しています。詳細は[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。
