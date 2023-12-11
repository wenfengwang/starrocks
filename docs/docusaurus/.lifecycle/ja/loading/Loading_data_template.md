---
displayed_sidebar: "Japanese"
unlisted: true
---

# \<SOURCE\>テンプレートからデータを読み込む

## テンプレートの手順

### スタイルについての注意

技術ドキュメントには、通常、ページ全体にリンクが散りばめられています。このドキュメントを見ると、他のページへのリンクがほとんどなく、リンクが存在するのはほとんどが**詳細情報**セクションの最後にあることに気づくかもしれません。すべてのキーワードを別のページにリンクさせる必要はありません。たとえば、`CREATE TABLE`が何を意味するかは読者が知っていると仮定し、知らない場合は検索バーをクリックして調べることができます。ドキュメントに注釈を入れることで、読者に他のオプションがあることを伝えることは問題ありません。詳細は**詳細情報**セクションに記載されています。このようにすることで、情報が必要な人々がそれを後で読むことができるようにします。

### テンプレート

このテンプレートは、Amazon S3からデータを読み込むプロセスに基づいており、一部は他のソースからの読み込みには適用されない場合があります。このテンプレートの流れに集中し、すべてのセクションを含める必要はないようにしてください。流れは次のとおりです:

#### 導入

このガイドに従うことで得られる結果を読者に伝える導入文です。S3ドキュメントの場合、最終的な結果は「S3からデータを非同期または同期のいずれかの方法で読み込むこと」です。

#### なぜ？

- 技術で解決されるビジネス上の問題の説明
- 使用される方法の利点と欠点（あれば）

#### データフローやその他の図

図や画像は役立ちます。複雑な技術を説明する際には、画像が役立つ場合があります。たとえば、データの分析にSupersetを使用する場合など、視覚的なものを生成する技術を説明する場合は、最終的な出力の画像を必ず含めてください。

データのフローが明らかでない場合はデータフロー図を使用して説明してください。このテンプレートでは、データの読み込みには2つの方法が記載されています。そのうちの1つはシンプルで、データフローセクションはありません。もう1つはより複雑です（StarRocksが複雑な作業を処理しており、ユーザーではありません！）、そして複雑なオプションにはデータフローセクションが含まれています。

#### 検証セクション付きの例

構文の詳細やその他の技術的な詳細よりも、例が先に来るべきです。多くの読者は、コピーして貼り付けて変更できる特定のテクニックを見つけるためにドキュメントにアクセスします。

可能であれば、動作する例を提供し、使用するデータセットを含めてください。このテンプレートの例では、AWSアカウントを持っており、キーとシークレットで認証できるユーザーなら誰でも使用できるS3に格納されたデータセットが使用されています。データセットを提供することで、例は読者にとってより価値があるものになります。

例が記述どおりに機能することを確認してください。これには2つのことが含まれます:

1. コマンドを記載された順序で実行していること
2. 必要な前提条件を含めていること。たとえば、例で`foo`データベースを参照している場合は、おそらく`CREATE DATABASE foo;`、`USE foo;`を行う必要があります。

検証は非常に重要です。説明しているプロセスには複数のステップが含まれる場合は、何かが達成されるべきタイミングで検証ステップを含めてください。これにより、読者が最後まで行ってみて、10番目のステップに誤植があることに気づくのを防ぐのに役立ちます。この例では、**進捗状況の確認**と`DESCRIBE user_behavior_inferred;`ステップが検証用です。

#### 詳細情報

テンプレートの最後には、主要な情報に加えて、メインボディで言及したオプションのリンクを含める場所があります。

### テンプレートに埋め込まれたノート

テンプレートのノートは、作業中にそれに注意を向けるために意図的にドキュメントのノートとは異なる形式でフォーマットされています。作業を進めるにつれて、**太字と斜体の注釈**を削除してください。

```markdown
***Note: 説明文***
```

## 最後に、テンプレートの開始

***Note: 複数の推奨選択肢がある場合は、読者にそのことをイントロで伝えてください。たとえば、S3からの読み込みの際、同期読み込みと非同期読み込みのオプションがあります:***

StarRocksでは、S3からデータを読み込むための2つのオプションが提供されます:

1. ブローカーロードを使用した非同期読み込み
2. `FILES()`テーブル関数を使用した同期読み込み

***Note: 読者に、どちらの選択肢を選ぶ理由を伝えてください:***

小規模データセットは、`FILES()`テーブル関数を使用して同期的に読み込まれ、大規模データセットはブローカーロードを使用して非同期的に読み込まれます。両方の方法には異なる利点があり、以下で説明します。

> **注記**
>
> StarRocksテーブルにデータを読み込むことは、そのStarRocksテーブルにINSERT権限を持つユーザーとしてのみ可能です。INSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)でINSERT権限を付与するための指示に従ってください。
`information_schema.loads`テーブルをクエリして進行状況を追跡します。複数の`LOAD`ジョブが実行されている場合は、ジョブに関連する`LABEL`でフィルタリングできます。以下の出力には、`user_behavior`のロードジョブに対する2つのエントリがあります。最初のレコードは`CANCELLED`状態を示し、出力の最後までスクロールすると、`listPath failed`が表示されます。2番目のレコードは成功を示し、有効なAWS IAMアクセスキーとシークレットがあります。

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

この時点でデータのサブセットも確認できます。

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

`FILES()`は、Parquetデータの列のデータ型を推測し、StarRocksテーブルのスキーマを生成できます。これにより、S3から直接`SELECT`でファイルをクエリするか、Parquetファイルスキーマに基づいてStarRocksが自動的にテーブルを作成できます。

> **注意**
>
> スキーマ推測はバージョン3.1の新機能であり、Parquet形式のみに提供され、ネストされたタイプはまだサポートされていません。

### 典型的な例

`FILES()`テーブル関数を使用した3つの例があります。

- S3から直接データをクエリする
- スキーマ推測を使用したテーブルの作成とロード
- テーブルを手動で作成し、次にデータをロードする

> **注意**
>
> これらの例で使用されるデータセットは、StarRocksアカウントのS3バケットにホストされています。有効な`aws.s3.access_key`と`aws.s3.secret_key`を使用できます。以下のコマンドの`AAA`と`BBB`を自分の資格情報に置き換えてください。

#### S3から直接クエリ

`FILES()`を使用してS3から直接クエリを行うことで、テーブルを作成する前にデータセットの内容をプレビューできます。例:

- データを保存せずにデータセットのプレビューを取得する
- 最小値と最大値をクエリし、使用するデータ型を決定する
- Nullをチェックする

```sql
SELECT * FROM FILES(
    "path" = "s3://starrocks-datasets/user_behavior_sample_data.parquet",
    "format" = "parquet",
    "aws.s3.region" = "us-east-1",
    "aws.s3.access_key" = "AAAAAAAAAAAAAAAAAAAA",
    "aws.s3.secret_key" = "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
) LIMIT 10;
```

> **注意**
>
> 列名はParquetファイルによって提供されていることに注目してください。

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

これは前の例の続きで、前のクエリはスキーマ推測を使用してテーブルの作成を自動化するために`CREATE TABLE`でラップされています。Parquetファイルのスキーマには列名と型が含まれているため、`FILES()`テーブル関数をParquetファイルとともに使用する場合、テーブルを作成する際に列名とタイプを指定する必要はありません。 StarRocksはスキーマを推測します。

> **注意**
>
> スキーマ推測を使用する際の`CREATE TABLE`の構文では、レプリカ数を設定することはできないため、テーブルを作成する前に設定してください。以下の例はシングルレプリカのシステム用です:
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
> 推測スキーマと手動作成のスキーマを比較する:
>
> - データ型
> - NULL可能性
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

挿入先のテーブルをカスタマイズすることができます:
- カラムのデータ型、NULL可能設定、またはデフォルト値
- キーの種類とカラム
- 分布
- その他

> **NOTE**
>
> 最も効率的なテーブル構造を作成するためには、データの使用方法とカラムの内容を知る必要があります。この文書ではテーブル設計はカバーされていませんが、ページの最後に**詳細情報**というリンクがあります。

この例では、Parquetファイルのデータに基づいてテーブルを作成しています。Parquetファイルのデータに関する知識はS3で直接ファイルをクエリすることで得ることができます。

- S3でファイルをクエリすることで、`Timestamp`カラムが`datetime`データ型に一致するデータを含んでいることが分かるため、次のDDLでカラムタイプが指定されています。
- S3のデータをクエリすることで、データセットにNULLの値がないことが分かるので、DDLは任意のカラムをNULLに設定しません。
- 期待されるクエリタイプの知識に基づいて、ソートキーとバケット化されたカラムが`UserID`に設定されています（データによっては、このデータに対してあなたの使用用途が異なる場合があり、ソートキーとして `UserID` の代わりに `ItemID` を使用することを選択するかもしれません）:

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

テーブルを作成したら、次のように `INSERT INTO` … `SELECT FROM FILES()` を使用してロードすることができます:

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

- 同期および非同期データのロードの詳細については、[データのロードの概要](../loading/Loading_intro.md) のドキュメントを参照してください。
- Broker Loadがロード中にデータ変換をサポートする方法については、[ロード時のデータ変換](../loading/Etl_in_loading.md) および [ロードを通じたデータの変更](../loading/Load_to_Primary_Key_tables.md) を参照してください。
- この文書ではIAMユーザーベースの認証のみをカバーしています。他のオプションについては、[AWSリソースへの認証](../integrations/authenticate_to_aws_resources.md) を参照してください。
- [AWS CLIコマンドリファレンス](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html) でS3 URIについて詳しく説明しています。
- [テーブル設計](../table_design/StarRocks_table_design.md) について詳しくはこちらを参照してください。
- Broker Loadには、上記の例に示すものよりも多くの設定オプションと使用オプションがあります。詳細は [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) にあります。