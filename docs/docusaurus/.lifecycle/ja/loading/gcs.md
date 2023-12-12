---
displayed_sidebar: "Japanese"
---

# GCSからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksではGCSからデータをロードするための2つのオプションが提供されています:

1. ブローカーロードを使用した非同期のロード
2. `FILES()` テーブル関数を使用した同期のロード

小さなデータセットは通常、`FILES()` テーブル関数を使用して同期的にロードされ、大きなデータセットは通常、ブローカーロードを使用して非同期的にロードされます。両方の方法には異なる利点があり、以下で説明します。

<InsertPrivNote />

## 接続の詳細を収集する

> **注意**
>
> これらの例ではサービスアカウントキー認証を使用しています。その他の認証方法については、このページの最下部にリンクがあります。
>
> このガイドでは、StarRocksがホストするデータセットを使用しています。このデータセットは認証済みのGCPユーザーなら誰でも読み取ることができるため、以下で使用する権限情報でParquetファイルを読み取ることができます。

GCSからデータをロードするには、以下が必要です：

- GCSバケット
- GCSオブジェクトキー(GCSバケット内の特定のオブジェクトにアクセスする場合)。オブジェクトキーには、GCSオブジェクトがサブフォルダに保存されている場合はプレフィックスを含めることができます。詳細な構文は**詳細情報**にリンクされています。
- GCSリージョン
- サービスアカウントアクセスキーとシークレット

## ブローカーロードの使用

非同期のブローカーロードプロセスは、GCSへの接続の作成、データの取得、およびデータのStarRocksへの保存を処理します。

### ブローカーロードの利点

- ブローカーロードはデータ変換、UPSERT、およびDELETE操作をサポートしています。
- ブローカーロードはバックグラウンドで実行され、ジョブを継続するためにクライアントが接続し続ける必要はありません。
- ブローカーロードは、実行のタイムアウトがデフォルトで4時間である長時間実行されるジョブに適しています。
- ParquetおよびORCファイルフォーマットに加えて、ブローカーロードはCSVファイルもサポートしています。

### データフロー

![ブローカーロードのワークフロー](../assets/broker_load_how-to-work_en.png)

1. ユーザーがロードジョブを作成します。
2. フロントエンド（FE）がクエリプランを作成し、そのプランをバックエンドノード（BE）に配布します。
3. バックエンド（BE）ノードがデータをソースから取得し、データをStarRocksにロードします。

### 典型的な例

テーブルを作成し、GCSからParquetファイルを取得してロードプロセスを開始し、データのロードの進行状況と成功を確認します。

> **注意**
>
> この文書の例では、CSVまたはORCファイルをロードしたい場合は、詳細情報にリンクされています。

#### テーブルの作成

テーブル用のデータベースを作成します：

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

テーブルを作成します。このスキーマは、StarRocksアカウントにホストされているGCSバケット内のサンプルデータセットと一致します。

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
> この文書の例では、`replication_num`プロパティが`1`に設定されているため、シンプルな単一のBEシステムで実行できます。3つ以上のBEを使用している場合は、DDLの`PROPERTIES`セクションを削除してください。

#### ブローカーロードを開始する

このジョブには4つの主要なセクションがあります：

- `LABEL`：`LOAD`ジョブの状態をクエリする際に使用される文字列。
- `LOAD`宣言：ソースURI、ディスティネーションテーブル、およびソースのデータフォーマット。
- `BROKER`：ソースの接続詳細。
- `PROPERTIES`：このジョブに適用するタイムアウト値やその他のプロパティ。

> **注意**
>
> これらの例で使用されているデータセットは、StarRocksアカウント内のGCSバケットにホストされています。有効なサービスアカウントの電子メール、キー、およびシークレットを使用できます。このオブジェクトは、GCP認証済みユーザーなら誰でも読み取ることができます。以下のコマンドでプレースホルダーの代わりに権限情報を置き換えてください。

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

進行状況を追跡するために`information_schema.loads`テーブルをクエリします。マルチプルの`LOAD`ジョブが実行されている場合は、ジョブに関連する`LABEL`でフィルタリングできます。以下の出力では、`user_behavior`のロードジョブについて2つのエントリがあります。最初のレコードは`CANCELLED`状態を示しており、「listPath failed」とのメッセージが表示されます。2番目のレコードはAWS IAMアクセスキーとシークレットが有効で成功したことを示しています。

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

この時点でデータのサブセットを確認することもできます。

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
```
## `FILES()` テーブル関数の使用

### `FILES()` の利点

`FILES()` は、Parquet データの列のデータ型を推論し、StarRocks テーブルのスキーマを生成することができます。これにより、S3 から `SELECT` を使用してファイルを直接クエリしたり、Parquet ファイルのスキーマに基づいて StarRocks が自動的にテーブルを作成したりする能力が提供されます。

> **注記**
>
> スキーマ推論は、バージョン 3.1 でのみ提供されており、Parquet フォーマットのみをサポートしており、ネストされた型はまだサポートされていません。

### 典型的な例

`FILES()` テーブル関数を使用した 3 つの典型的な例があります。

- S3 からデータを直接クエリする
- スキーマ推論を使用したテーブルの作成とロード
- 手動でテーブルを作成してからデータをロードする

#### S3 から直接クエリ

`FILES()` を使用して S3 から直接クエリすることで、テーブルを作成する前にデータセットのコンテンツを良くプレビューできます。たとえば:

- データを保存せずにデータセットのプレビューを取得
- 最小値と最大値をクエリし、使用するデータ型を決定
- Null の確認

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
> 列名は Parquet ファイルによって提供されることに注意してください。

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

これは前の例の続きで、前のクエリを `CREATE TABLE` でラップして、スキーマ推論を使用してテーブル作成を自動化します。Parquet フォーマットは列名と型が含まれているため、`FILES()` テーブル関数を使用してテーブルを作成する際には、列名と型を指定する必要はありません。

> **注記**
>
> スキーマ推論を使用する場合、`CREATE TABLE` の構文ではレプリカ数の設定は許可されていません。したがって、テーブルを作成する前に設定してください。以下の例はシステムが単一のレプリカを持つ場合のものです:
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

> **注記**
>
> 推定されたスキーマを手動で作成したスキーマと比較してください:
>
> - データ型
> - Null 可能性
> - キーのフィールド

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

#### 既存のテーブルへのロード

挿入先のテーブルをカスタマイズすることができます。たとえば:

- 列のデータ型、Null 設定、デフォルト値
- キータイプと列
- 分散
- その他

> **注記**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法や列の内容を知る必要があります。このドキュメントではテーブル設計についてはカバーしていませんが、ページの最後の**詳細情報**にリンクがあります。

この例では、Parquet ファイル内のデータをクエリしてデータベースから取得し、以下の DDL で指定される列タイプに一致するデータが `Timestamp` 列に含まれていることがわかります。

- S3 のデータをクエリすることで、データセットに Null 値がないことがわかります。したがって、DDL ではどの列も Null として設定されていません。
- 予想されるクエリタイプの知識に基づいて、ソートキーとバケット化された列は `UserID` に設定されています (このデータについては使用用途が異なるかもしれません)。

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

テーブルを作成した後、`INSERT INTO` … `SELECT FROM FILES()` でロードできます:

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
```markdown
    + {R}
    + {R}
  + {R}
+ {R}
```

## 詳細情報

- このドキュメントではサービスアカウントキー認証のみを取り扱っています。その他のオプションについては[Google Cloud Storageリソースへの認証](../integrations/authenticate_to_gcs.md)を参照してください。
- 同期および非同期のデータローディングの詳細については、[データローディングの概要](../loading/Loading_intro.md)文書を参照してください。
- ブローカーロードがローディング中にデータ変換をサポートする方法については、[ローディング時にデータを変換](../loading/Etl_in_loading.md)および[ローディングを通じてデータを変更](../loading/Load_to_Primary_Key_tables.md)でご確認ください。
- [テーブル設計](../table_design/StarRocks_table_design.md)の詳細についてはこちらを参照してください。
- 上記の例に含まれない、Broker Loadの多くの構成および使用オプションの詳細については、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)をご覧ください。