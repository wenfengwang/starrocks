---
displayed_sidebar: English
unlisted: true
---

# \<SOURCE\> TEMPLATE からデータをロードする

## テンプレートの指示

### スタイルについての注意

技術文書には通常、他の文書へのリンクが多数含まれています。このドキュメントを見ると、ページからのリンクが少なく、ほとんどのリンクがドキュメントの下部にある**詳細情報**セクションにあることに気付くでしょう。すべてのキーワードを別のページにリンクする必要はありません。読者が`CREATE TABLE`の意味を知っていると仮定し、知らない場合は検索バーで調べることができると考えてください。ドキュメントに注記を入れて、他のオプションがあり、詳細は**詳細情報**セクションで説明されていることを読者に伝えることは問題ありません。これにより、情報が必要な人は、手元のタスクを完了した***後に***読むことができます。

### テンプレート

このテンプレートは、Amazon S3からデータをロードするプロセスに基づいていますが、
一部は他のソースからのロードには適用されません。このテンプレートの流れに集中し、すべてのセクションを含めることについて心配しないでください。意図されたフローは以下の通りです：

#### イントロダクション

このガイドに従った場合の最終結果について読者に知らせる導入文。S3のドキュメントの場合、最終結果は「非同期または同期のいずれかの方法でS3からデータをロードすること」です。

#### なぜ？

- この技術で解決されるビジネス問題の説明
- 説明されている方法の利点と欠点（あれば）

#### データフローまたはその他の図

図や画像は役立つことがあります。複雑な技術を説明していて、画像が役立つ場合は使用してください。視覚的なものを生成する技術（例えば、Supersetを使用してデータを分析する場合）を説明している場合は、終了製品の画像を必ず含めてください。

フローが自明でない場合は、データフロー図を使用してください。コマンドがStarRocksに複数のプロセスを実行させ、それらのプロセスの出力を組み合わせてからデータを操作する場合は、データフローの説明が必要かもしれません。このテンプレートでは、データロードのための2つの方法が説明されています。一つはシンプルでデータフローセクションはありませんが、もう一つはより複雑です（StarRocksが複雑な作業を処理します、ユーザーではありません！）、そして複雑なオプションにはデータフローセクションが含まれています。

#### 検証セクション付きの例

例は、構文の詳細やその他の深い技術的詳細の前に来るべきです。多くの読者は、コピーして貼り付けて修正できる特定の技術を見つけるためにドキュメントを訪れます。

可能であれば、実際に機能する例とそれに使用するデータセットを提供してください。このテンプレートの例では、AWSアカウントを持ち、キーとシークレットで認証できるすべてのユーザーが使用できるS3に保存されたデータセットを使用します。データセットを提供することで、例は読者にとってより価値があります。なぜなら、彼らは説明された技術を完全に体験できるからです。

例が記述された通りに機能することを確認してください。これは2つのことを意味します：

1. 示された順序でコマンドを実行しました
2. 必要な前提条件を含めました。例えば、例がデータベース`foo`を参照している場合、おそらくそれに先立って`CREATE DATABASE foo;`、`USE foo;`が必要です。

検証は非常に重要です。説明しているプロセスに複数のステップが含まれている場合、何かが達成されるべきであるたびに検証ステップを含めてください。これにより、読者が最後まで進んで手順10でタイプミスがあったことに気づかないようにするのに役立ちます。この例では、**進捗を確認**すると`DESCRIBE user_behavior_inferred;`のステップは検証のためです。

#### 詳細情報

テンプレートの最後には、本文で言及したオプショナル情報を含む関連情報へのリンクを置く場所があります。

### テンプレートに埋め込まれたノート

テンプレートのノートは、ドキュメントのノートとは異なる形式で意図的に書式設定されており、テンプレートを作業する際にあなたの注意を引くようになっています。進行中に太字イタリックのノートを削除してください：

```markdown
***注記: 説明的なテキスト***
```

## 最後に、テンプレートの開始

***注記: 複数の推奨選択肢がある場合は、
イントロで読者に伝えてください。例えば、S3からのロード時には、
同期ロードと非同期ロードのオプションがあります。***

StarRocksはS3からデータをロードするための2つのオプションを提供します：

1. Broker Loadを使用した非同期ロード
2. `FILES()`テーブル関数を使用した同期ロード

***注記: 読者に、なぜ一方の選択肢を他方より選ぶべきかを伝えてください。***

通常、小規模なデータセットは`FILES()`テーブル関数を使用して同期的にロードされ、大規模なデータセットはBroker Loadを使用して非同期にロードされます。これら2つの方法は異なる利点を持ち、以下で説明されます。

> **注記**
>
> StarRocksテーブルにデータをロードするには、そのStarRocksテーブルにINSERT権限を持つユーザーである必要があります。INSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)に記載されている手順に従って、StarRocksクラスタに接続するために使用するユーザーにINSERT権限を付与してください。

## Broker Loadの使用

非同期のBroker Loadプロセスは、S3への接続を確立し、データを取得し、StarRocksにデータを格納する作業を処理します。

### Broker Loadの利点

- Broker Loadは、ロード中にデータ変換、UPSERT、DELETE操作をサポートします。
- Broker Loadはバックグラウンドで実行され、クライアントはジョブが続行されるために接続されている必要はありません。
- 長時間実行されるジョブにはBroker Loadが推奨され、デフォルトのタイムアウトは4時間です。
- ParquetとORCファイル形式に加えて、Broker LoadはCSVファイルもサポートします。

### データフロー

***注記: 複数のコンポーネントやステップを含むプロセスは、図を使うと理解しやすくなることがあります。この例には、ユーザーがBroker Loadオプションを選択したときに発生するステップを説明するのに役立つ図が含まれています。***

![Broker Loadのワークフロー](../assets/broker_load_how-to-work_en.png)

1. ユーザーがロードジョブを作成します。

2. フロントエンド(FE)はクエリプランを作成し、そのプランをバックエンドノード(BE)に配布します。
3. バックエンド(BE)ノードはソースからデータを取得し、StarRocksにデータをロードします。

### 典型的な例

テーブルを作成し、S3からParquetファイルをプルするロードプロセスを開始し、データロードの進行状況と成功を確認します。

> **注記**
>
> この例ではParquet形式のサンプルデータセットを使用しています。CSVまたはORCファイルをロードしたい場合は、このページの下部にリンクされている情報を参照してください。

#### テーブルの作成

テーブル用のデータベースを作成します：

```SQL
CREATE DATABASE IF NOT EXISTS project;
USE project;
```

テーブルを作成します。このスキーマは、StarRocksアカウントでホストされているS3バケット内のサンプルデータセットに対応しています。

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

#### 接続情報の収集

> **注記**
>
> この例ではIAMユーザーベースの認証を使用しています。他の認証方法については、このページの下部にリンクがあります。

S3からデータをロードするには、以下が必要です：

- S3バケット
- S3オブジェクトキー（オブジェクト名）（バケット内の特定のオブジェクトにアクセスする場合）。オブジェクトキーには、S3オブジェクトがサブフォルダーに保存されている場合、プレフィックスを含めることができます。完全な構文は**詳細情報**でリンクされています。
- S3リージョン
- アクセスキーとシークレットキー

#### Broker Loadの開始

このジョブには4つの主要なセクションがあります：

- `LABEL`：`LOAD`ジョブの状態を照会する際に使用される文字列。
- `LOAD`宣言：ソースURI、宛先テーブル、およびソースデータ形式。
- `BROKER`：ソースへの接続情報。
- `PROPERTIES`：タイムアウト値とこのジョブに適用するその他のプロパティ。

> **注記**
>
> これらの例で使用されているデータセットはStarRocksアカウントのS3バケットにホストされています。任意の有効な`aws.s3.access_key`と`aws.s3.secret_key`を使用できますが、オブジェクトはAWS認証ユーザーなら誰でも読み取り可能です。以下のコマンドで`AAA`と`BBB`をあなたの認証情報に置き換えてください。

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

#### 進捗の確認

`information_schema.loads`テーブルをクエリして進捗を追跡します。複数の`LOAD`ジョブが実行されている場合は、関連する`LABEL`でフィルタリングできます。以下の出力には`user_behavior`のロードジョブに関する2つのエントリがあります。最初のレコードは`CANCELLED`の状態を示し、出力の最後には`listPath failed`と表示されます。2番目のレコードは有効なAWS IAMアクセスキーとシークレットを使用して成功したことを示しています。

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
SELECT * FROM user_behavior LIMIT 10;
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

`FILES()`はParquetデータの列のデータ型を推測し、StarRocksテーブルのスキーマを生成することができます。これにより、S3から直接ファイルに対して`SELECT`クエリを実行したり、Parquetファイルのスキーマに基づいてStarRocksにテーブルを自動的に作成させたりすることが可能になります。

> **注記**
>
> スキーマ推測はバージョン3.1の新機能で、現在はParquet形式のみで提供されており、ネストされた型はまだサポートされていません。

### 典型的な例

`FILES()`テーブル関数を使用した3つの例：

- S3から直接データをクエリする
- スキーマ推測を使用してテーブルを作成し、データをロードする
- 手動でテーブルを作成し、その後データをロードする

> **注記**
>
> これらの例で使用されているデータセットは、StarRocksアカウントのS3バケットにホストされています。任意の有効な `aws.s3.access_key` と `aws.s3.secret_key` は、AWS認証されたユーザーであれば誰でも読み取り可能なオブジェクトですので、使用できます。以下のコマンドで `AAA` と `BBB` をあなたの資格情報に置き換えてください。

#### S3から直接クエリする

`FILES()` を使用してS3から直接クエリすることで、テーブルを作成する前にデータセットの内容を良くプレビューできます。例えば：

- データを保存せずにデータセットのプレビューを取得する。
- 最小値と最大値をクエリして、使用するデータ型を決定する。
- NULL値をチェックする。

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

#### スキーマ推論を用いたテーブル作成

これは前の例の続きで、`CREATE TABLE` を使用してスキーマ推論を用いたテーブル作成を自動化しています。Parquetファイルを使用する `FILES()` テーブル関数では、Parquet形式が列名と型を含んでおり、StarRocksがスキーマを推測するため、列名と型を指定する必要はありません。

> **注記**
>
> スキーマ推論を使用する `CREATE TABLE` の構文では、レプリカ数を設定することはできませんので、テーブル作成前に設定してください。以下の例は、シングルレプリカシステム用です：
>
> `ADMIN SET FRONTEND CONFIG ('default_replication_num' = "1");`

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
> 推測されたスキーマと手作業で作成されたスキーマを比較してください：
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

挿入するテーブルをカスタマイズすることができます。例えば：

- 列のデータ型、NULL許容設定、またはデフォルト値
- キーの種類と列
- 分散
- など

> **注記**
>
> 最も効率的なテーブル構造を作成するには、データの使用方法と列の内容に関する知識が必要です。このドキュメントではテーブル設計については触れていませんが、ページの最後に**さらなる情報**へのリンクがあります。

この例では、テーブルがどのようにクエリされるか、およびParquetファイル内のデータに関する知識に基づいてテーブルを作成しています。Parquetファイル内のデータに関する知識は、S3でファイルを直接クエリすることで得られます。

- S3でのファイルのクエリは、`Timestamp` 列が `datetime` データ型に一致するデータを含んでいることを示しているため、列の型は以下のDDLで指定されます。
- S3でデータをクエリすると、データセットにNULL値がないことがわかるので、DDLではどの列もNULL許容として設定しません。
- 想定されるクエリタイプに基づいて、ソートキーとバケット列は `UserID` 列に設定されます（このデータに対しては、ソートキーとして `ItemID` を `UserID` に加えて、または代わりに使用することを決定するかもしれません）。

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

テーブルを作成した後、`INSERT INTO` … `SELECT FROM FILES()` を使用してロードできます：

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

- 同期および非同期のデータロードの詳細については、[データロードの概要](../loading/Loading_intro.md)のドキュメントをご覧ください。
- ブローカーロードがロード中にデータ変換をサポートする方法については、[ロード時のデータ変換](../loading/Etl_in_loading.md)と[ロードを通じたデータの変更](../loading/Load_to_Primary_Key_tables.md)をご覧ください。
- このドキュメントではIAMユーザーベースの認証のみをカバーしています。他のオプションについては、[AWSリソースへの認証](../integrations/authenticate_to_aws_resources.md)をご覧ください。
- [AWS CLI コマンドリファレンス](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html)では、S3 URIについて詳細に説明しています。
- [テーブル設計](../table_design/StarRocks_table_design.md)についてもっと学びましょう。
- Broker Loadは上記の例に示されるものよりも多くの設定オプションと使用オプションを提供しますが、詳細は[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)に記載されています。
