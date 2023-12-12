---
displayed_sidebar: "Japanese"
unlisted: true
---

# \<SOURCE\>テンプレートからデータをロードする

## テンプレートの手順

### スタイルについての注意

技術文書には通常、さまざまなドキュメントへのリンクがあります。このドキュメントを見ると、ページからリンクが少なく、ほとんどのリンクが**詳細情報**セクションの一番下にあることに気づくかもしれません。すべてのキーワードを別のページにリンクさせる必要はありません。`CREATE TABLE`という用語を読者が知っていると想定し、知らない場合は検索バーをクリックして調べることができると仮定してください。ドキュメントには別のオプションがあることを読者に伝えるためにノートを記述することは問題ありません。これにより、情報が必要な人が後で読むことができるようになります。

### テンプレート

このテンプレートはAmazon S3からデータをロードするプロセスに基づいており、それ以外のソースからのロードには適用されない部分もあります。このテンプレートの流れに集中し、すべてのセクションを含める必要はないでしょう。以下の流れを意識してください:

#### イントロダクション

このガイドに従うことで得られる結果を読者に伝える紹介的な文章。S3のドキュメントの場合、結果は「S3からデータを非同期または同期の方法で取得すること」です。

#### なぜ？

- 技術手法で解決されるビジネス上の問題の説明
- 記述されている手法の利点および欠点（あれば）

#### データフローやその他の図

図や画像は役立つ場合があります。複雑な手法を説明する場合には、画像が役立ちます。視覚的なものを生成する手法を説明する場合（たとえば、Supersetを使用してデータを分析する場合）、最終成果の画像を必ず含めてください。

データフローが非常に明らかでない場合は、データフロー図を使用してください。このテンプレートでは、データのロード方法が2つあります。1つは単純でデータフローセクションがなく、もう1つは（複雑な作業をStarRocksが処理しており、ユーザーではありません！）複雑なオプションがあり、この複雑なオプションにはデータフローセクションが含まれています。

#### 検証セクションを含む例

構文の詳細やその他の技術的な詳細よりも前に例を示すことが望ましいです。多くの読者は、コピーして貼り付けて変更できる特定の技法を見つけるためにこのドキュメントに来ることがあります。

可能であれば、動作する例とデータセットを含めてください。このテンプレートの例では、AWSアカウントを持っており、キーとシークレットで認証できる誰でもが使用できるS3に格納されたデータセットを使用しています。データセットを提供することで、例が読者にとって価値のあるものになります。

例がそのまま動作することを確認してください。これには2つのことを含みます:

1. コマンドを提示された順に実行したこと
2. 必要な前提条件が含まれていること。例えば、例が`foo`データベースを参照する場合、おそらく`CREATE DATABASE foo;`、`USE foo;`の前にそれを前提条件として記述する必要があります。

検証は非常に重要です。説明しているプロセスに複数のステップが含まれている場合は、何かが達成されるべきタイミングごとに検証ステップを含めることで、読者が最後になってステップ10でタイプミスをしたことに気付くのを回避するのに役立ちます。この例では、**進捗を確認**および`DESCRIBE user_behavior_inferred;`ステップが検証用です。

#### 詳細情報

テンプレートの最後には、メインの本文で言及した関連情報へのリンクを含む場所があります。オプションの情報へのリンクも含まれます。

### テンプレートに埋め込まれたノート

テンプレートのノートは、テンプレートを作業する際に注意を引くために意図的にドキュメントのノートのフォーマットとは異なるようにフォーマットされています。作業中に太字の斜体のノートを削除してください:

```markdown
***Note: 説明的なテキスト***
```

## 最後に、テンプレートの開始

***Note: 複数の推奨選択肢がある場合、イントロで読者にそれを伝えてください。たとえば、S3からのロードの場合、同期的なロードと非同期的なロードの2つのオプションがあります:***

StarRocksでは、S3からデータをロードするための2つのオプションが提供されています:

1. Broker Loadを使用した非同期ロード
2. `FILES()`テーブル関数を使用した同期ロード

***Note: 読者がなぜ1つの選択肢を他よりも選ぶべきかを伝えてください:***

小さなデータセットは、`FILES()`テーブル関数を使用して同期的にロードされることが多く、大きなデータセットはBroker Loadを使用して非同期的にロードされることが多いです。これら2つの方法には異なる利点があり、以下で説明します。

> **NOTE**
>
> StarRocksのテーブルにデータをロードできるのは、そのStarRocksテーブルにINSERT権限を持っているユーザーのみです。INSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)で挙げられた指示に従ってINSERT権限をユーザーに付与してください。

## Broker Loadの使用

非同期のBroker Loadプロセスは、S3への接続を行い、データを取り込み、そしてデータをStarRocksに保存します。

### Broker Loadの利点

- Broker Loadは、データ変換、UPSERT、およびDELETE操作をサポートしています。
- Broker Loadはバックグラウンドで実行され、ジョブを継続するためにクライアントが接続を維持する必要はありません。
- Broker Loadは長時間実行されるジョブに適しており、デフォルトのタイムアウトは4時間です。
- ParquetやORCファイル形式に加えて、Broker LoadはCSVファイルもサポートしています。

### データフロー

***Note: 複数のコンポーネントやステップを含むプロセスは、図を使用すると理解しやすい場合があります。この例には、ユーザーがBroker Loadオプションを選択した際に発生するステップを説明する図が含まれています。***

![Broker Loadのワークフロー](../assets/broker_load_how-to-work_jp.png)

1. ユーザーはロードジョブを作成します。
2. フロントエンド（FE）がクエリプランを作成し、プランをバックエンドノード（BE）に配信します。
3. バックエンド（BE）ノードがソースからデータを取得し、データをStarRocksにロードします。

### 典型的な例

テーブルを作成し、S3からParquetファイルを取得してロードプロセスを開始し、データのロードの進捗と成功を検証します。

> **注意**
>
> これらの例では、Parquet形式のサンプルデータセットを使用しています。CSVまたはORCファイルをロードしたい場合は、このページの一番下にリンクされている情報をご覧ください。

#### テーブルを作成

テーブルのためにデータベースを作成してください:

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

テーブルを作成します。このスキーマは、StarRocksアカウントにホストされているS3バケット内のサンプルデータセットと一致します。

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

#### 接続の詳細を収集

> **注意**
>
> これらの例ではIAMユーザーベースの認証を使用しています。その他の認証方法については、このページの一番下にリンクがあります。

S3からデータをロードするには、次の情報が必要です:

- S3バケット
- S3オブジェクトキー（オブジェクト名）。バケット内の特定のオブジェクトにアクセスする場合はオブジェクトキーを指定します。オブジェクトキーにはプレフィックスが含まれることがあります。完全な構文は**詳細情報**にリンクされています。
- S3リージョン
- アクセスキーとシークレット

#### Broker Loadを開始

このジョブには4つのメインセクションがあります:

- `LABEL`: `LOAD`ジョブのステータスをクエリする際に使用される文字列。
- `LOAD`宣言: ソースURI、宛先テーブル、およびソースデータの形式。
- `BROKER`: ソースの接続詳細。
- `PROPERTIES`: このジョブに適用するタイムアウト値やその他のプロパティ。

> **注意**
>
> これらの例では、StarRocksアカウント内のS3バケットにホストされているデータセットが使用されます。AWS認証ユーザーであればどのような有効な`aws.s3.access_key`と`aws.s3.secret_key`でも使用できます。コマンドの`AAA`と`BBB`の部分に自分の認証情報を置き換えてください。

```SQL
LOAD LABEL user_behavior
(
    DATA INFILE("s3://starrocks-datasets/user_behavior_sample_data.parquet")
    INTO TABLE user_behavior
    FORMAT AS "parquet"
 )
 WITH BROKER
 (
    "aws.s3.enable_ssl" = "true",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
 )
PROPERTIES
(
    "timeout" = "72000"
);
```

#### 進捗を確認
`information_schema.loads`テーブルをクエリして進捗状況を追跡します。複数の`LOAD`ジョブが実行中の場合は、ジョブに関連付けられた`LABEL`でフィルタリングできます。以下の出力では、`user_behavior`のロードジョブに2つのエントリがあります。最初のレコードは`CANCELLED`の状態を示し、出力の最後までスクロールすると`listPath failed`と表示されます。2番目のレコードは有効なAWS IAMアクセスキーとシークレットを使用して成功を示しています。

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

この時点でデータのサブセットを確認できます。

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

`FILES()`は、Parquetデータの列のデータ型を推論し、StarRocksテーブルのスキーマを生成できます。これにより、`SELECT`を使用してS3からファイルを直接クエリしたり、Parquetファイルスキーマに基づいてStarRocksが自動的にテーブルを作成したりできます。

> **注記**
>
> スキーマ推論はバージョン3.1の新機能であり、Parquet形式のみをサポートしており、ネストされたタイプはまだサポートされていません。

### 典型的な例

`FILES()`テーブル関数を使用した3つの例があります：

- S3からデータを直接クエリ
- スキーマ推論を使用したテーブルの作成とロード
- テーブルの手動作成とデータのロード

> **注記**
>
> これらの例で使用されているデータセットは、StarRocksアカウント内のS3バケットにホストされています。有効な`aws.s3.access_key`と`aws.s3.secret_key`は、任意のAWS認証ユーザーによって読み取り可能なオブジェクトで使用できます。以下のコマンドで`AAA`と`BBB`の資格情報を置き換えてください。

#### S3から直接クエリ

`FILES()`を使用してS3から直接クエリすることで、テーブルを作成する前にデータセットの内容をプレビューできます。例：

- データを保存せずにデータセットのプレビューを取得
- 最小値と最大値をクエリして使用するデータ型を決定
- NULLをチェック

```sql
SELECT * FROM FILES(
    "path" = "s3://starrocks-datasets/user_behavior_sample_data.parquet",
    "format" = "parquet",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
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

これは前の例の続きで、前のクエリはスキーマ推論を使用したテーブル作成でラップされています。Parquetファイルの形式には列名とタイプが含まれており、StarRocksはスキーマを推論します。

> **注記**
>
> スキーマ推論を使用する場合、`CREATE TABLE`の構文ではレプリカ数の設定は許可されていないため、テーブルを作成する前に設定してください。以下は、単一のレプリカを持つシステムの例です：
>
> `ADMIN SET FRONTEND CONFIG ('default_replication_num' ="1");`

```sql
CREATE DATABASE IF NOT EXISTS project;
USE project;

CREATE TABLE `user_behavior_inferred` AS
SELECT * FROM FILES(
    "path" = "s3://starrocks-datasets/user_behavior_sample_data.parquet",
    "format" = "parquet",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
);
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

> **NOTE**
>
> 推定されるスキーマと手動で作成されたスキーマを比較してください：
>
> - データ型
> - NULL可能
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

#### 既存のテーブルにローディングする

挿入先のテーブルをカスタマイズすることができます。たとえば以下のようなものです：

- カラムのデータ型、NULLの設定、またはデフォルト値
- キータイプとカラム
- 分布
- その他

> **NOTE**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法とカラムの内容を知る必要があります。この文書ではテーブル設計についてカバーしていませんが、ページの最後に「詳細情報」のリンクがあります。

この例では、Parquetファイルのデータがクエリされる方法とテーブルのデータに基づいてテーブルを作成しています。Parquetファイルのデータの知識は、S3でファイルを直接クエリすることで得ることができます。

- S3でのファイルのクエリから、`Timestamp`カラムが `datetime` データ型に一致するデータを含むことが示されるため、以下のDDLでカラムのタイプが指定されています。
- S3のデータをクエリすることで、データセットにNULL値がないことが分かるため、DDLはいかなるカラムもnullableとしていません。
- 予想されるクエリの種類の知識に基づいて、ソートキーとバケティングカラムがカラム `UserID` に設定されています（このデータに対してお使いの用途によっては、このデータのために `UserID` の代わりに `ItemID` をソートキーとして使用することを決定するかもしれません。:

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

テーブルを作成した後、以下のようにしてデータをロードできます：`INSERT INTO` … `SELECT FROM FILES()`:

```SQL
INSERT INTO user_behavior_declared
  SELECT * FROM FILES(
    "path" = "s3://starrocks-datasets/user_behavior_sample_data.parquet",
    "format" = "parquet",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
);
```

## 詳細情報

- 同期および非同期データローディングの詳細については、[データローディングの概要](../loading/Loading_intro.md) のドキュメントを参照してください。
- Broker Load がローディング中にデータの変換をサポートする方法については、[ローディング時のデータ変換](../loading/Etl_in_loading.md) および [ローディングを通じたデータの変更](../loading/Load_to_Primary_Key_tables.md) を参照してください。
- この文書はIAMユーザーベースの認証のみを扱っています。その他のオプションについては、[AWSリソースへの認証](../integrations/authenticate_to_aws_resources.md) を参照してください。
- [AWS CLI コマンドリファレンス](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html) はS3 URIについて詳しく説明しています。
- [テーブル設計](../table_design/StarRocks_table_design.md) について詳しく学ぶ。
- Broker Load には上記の例よりも多くの構成および使用オプションがあります。詳細については、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) を参照してください。