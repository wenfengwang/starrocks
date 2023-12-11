---
displayed_sidebar: "Japanese"
---

# INSERTを使用してデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

このトピックでは、SQLステートメントINSERTを使用してStarRocksにデータをロードする方法について説明します。

MySQLや多くの他のデータベース管理システムと同様に、StarRocksはINSERTを使用して内部テーブルにデータをロードすることをサポートしています。 VALUES句を使用して1行以上のデータを直接挿入して、機能やデモをテストすることができます。また、クエリの結果によって定義されたデータを内部テーブルに挿入することもできます。StarRocks v3.1以降では、INSERTコマンドおよびテーブル関数FILES（）を使用してクラウドストレージ上のファイルからデータを直接ロードすることができます。

StarRocks v2.4では、INSERT OVERWRITEを使用してテーブルにデータを上書きすることもサポートされています。 INSERT OVERWRITEステートメントは、上書き機能を実装するために、次の操作を統合しています。

1.元のデータを保存しているパーティションに基づいて一時パーティションを作成します。
2.データを一時パーティションに挿入します。
3.元のパーティションと一時的なパーティションを交換します。

> **注**
>
> 上書き前にデータを検証する必要がある場合は、INSERT OVERWRITEを使用せずに、パーティションを交換する前にデータを上書きし、検証するための手順に従うことができます。

## 用心事

- MySQLクライアントから同期INSERTトランザクションをキャンセルするには、**Ctrl** と **C** キーを押してください。
- [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を使用して非同期INSERTタスクを送信できます。
- StarRocksの現行バージョンでは、データのいずれかの行がテーブルのスキーマに準拠していない場合、INSERTトランザクションはデフォルトで失敗します。たとえば、テーブルのマッピングフィールドの長さを超える場合、行の長さが制限を超える場合、INSERTトランザクションは失敗します。セッション変数`enable_insert_strict`を`false`に設定して、テーブルに一致しない行をフィルタリングしてトランザクションを続行できます。
- StarRocksに頻繁にINSERTステートメントを実行して、データを小さなバッチでStarRocksにロードすると、過剰なデータバージョンが生成されます。クエリのパフォーマンスに深刻な影響を与えます。本番環境では、頻繁にINSERTコマンドを使用してデータをロードすることは避け、日常的なデータロードのルーチンとして使用することは推奨しません。アプリケーションや解析シナリオによっては、ストリーミングデータや小さなデータバッチをロードするソリューションが求めらる場合があります。このような場合は、データソースとしてApache Kafka®を使用し、[Routine Load](../loading/RoutineLoad.md)を介してデータをロードすることをお勧めします。
- INSERT OVERWRITEステートメントを実行する場合、StarRocksは元のデータを保存しているパーティションに一時パーティションを作成し、新しいデータを一時パーティションに挿入し、[元のパーティションを一時的なパーティションと交換します](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#use-a-temporary-partition-to-replace-current-partition)。これらの操作はすべてFEリーダーノードで実行されます。したがって、FEリーダーノードがINSERT OVERWRITEコマンドの実行中にクラッシュすると、ロードトランザクション全体が失敗し、一時パーティションは切り捨てられます。

## 準備

### 権限の確認

<InsertPrivNote />

### オブジェクトの作成

`load_test`という名前のデータベースを作成し、宛先テーブルとして `insert_wiki_edit`、ソーステーブルとして`source_wiki_edit`を作成します。

> **注**
>
> このトピックで示される例は、`insert_wiki_edit`テーブルと`source_wiki_edit`テーブルを基にしています。独自のテーブルとデータで作業したい場合は、準備をスキップして次のステップに進むことができます。

```SQL
CREATE DATABASE IF NOT EXISTS load_test;
USE load_test;
CREATE TABLE insert_wiki_edit
(
    event_time      DATETIME,
    channel         VARCHAR(32)      DEFAULT '',
    user            VARCHAR(128)     DEFAULT '',
    is_anonymous    TINYINT          DEFAULT '0',
    is_minor        TINYINT          DEFAULT '0',
    is_new          TINYINT          DEFAULT '0',
    is_robot        TINYINT          DEFAULT '0',
    is_unpatrolled  TINYINT          DEFAULT '0',
    delta           INT              DEFAULT '0',
    added           INT              DEFAULT '0',
    deleted         INT              DEFAULT '0'
)
DUPLICATE KEY(
    event_time,
    channel,
    user,
    is_anonymous,
    is_minor,
    is_new,
    is_robot,
    is_unpatrolled
)
PARTITION BY RANGE(event_time)(
    PARTITION p06 VALUES LESS THAN ('2015-09-12 06:00:00'),
    PARTITION p12 VALUES LESS THAN ('2015-09-12 12:00:00'),
    PARTITION p18 VALUES LESS THAN ('2015-09-12 18:00:00'),
    PARTITION p24 VALUES LESS THAN ('2015-09-13 00:00:00')
)
DISTRIBUTED BY HASH(user);

CREATE TABLE source_wiki_edit
(
    event_time      DATETIME,
    channel         VARCHAR(32)      DEFAULT '',
    user            VARCHAR(128)     DEFAULT '',
    is_anonymous    TINYINT          DEFAULT '0',
    is_minor        TINYINT          DEFAULT '0',
    is_new          TINYINT          DEFAULT '0',
    is_robot        TINYINT          DEFAULT '0',
    is_unpatrolled  TINYINT          DEFAULT '0',
    delta           INT              DEFAULT '0',
    added           INT              DEFAULT '0',
    deleted         INT              DEFAULT '0'
)
DUPLICATE KEY(
    event_time,
    channel,user,
    is_anonymous,
    is_minor,
    is_new,
    is_robot,
    is_unpatrolled
)
PARTITION BY RANGE(event_time)(
    PARTITION p06 VALUES LESS THAN ('2015-09-12 06:00:00'),
    PARTITION p12 VALUES LESS THAN ('2015-09-12 12:00:00'),
    PARTITION p18 VALUES LESS THAN ('2015-09-12 18:00:00'),
    PARTITION p24 VALUES LESS THAN ('2015-09-13 00:00:00')
)
DISTRIBUTED BY HASH(user);
```

> **注意**
>
> v2.5.7以降、StarRocksは、テーブルを作成したりパーティションを追加したりする際に、バケツ（BUCKETS）の数を自動的に設定できます。バケツの数を手動で設定する必要はもはやありません。詳細については、[バケツの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

## INSERT INTO VALUESを使用してデータを挿入

INSERT INTO VALUESコマンドを使用して、特定のテーブルに1行以上のデータを追加することができます。複数行はコンマ（,）で区切られます。詳細な手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

> **注意**
>
> INSERT INTO VALUESを使用してデータを挿入するのは、小さなデータセットでDEMOを検証する必要がある場合にのみ適用されます。大規模なテストまたは本番環境では推奨されません。StarRocksに大量データをロードするには、シナリオに合った他のオプションを利用できるかどうかを確認するために、[Ingestion Overview](../loading/Loading_intro.md)を参照してください。

次の例では、`source_wiki_edit`データソーステーブルに`insert_load_wikipedia`ラベルを付けて2行を挿入します。ラベルは、データベース内の各データロードトランザクションのユニークな識別ラベルです。

```SQL
INSERT INTO source_wiki_edit
WITH LABEL insert_load_wikipedia
VALUES
    ("2015-09-12 00:00:00","#en.wikipedia","AustinFF",0,0,0,0,0,21,5,0),
    ("2015-09-12 00:00:00","#ca.wikipedia","helloSR",0,1,0,1,0,3,23,0);
```

## SELECT INTO INSERTを使用してデータを挿入

SELECT INTO INSERTコマンドを使用して、データソーステーブルのクエリ結果をターゲットテーブルにロードすることができます。SELECT INTO INSERTコマンドは、データソーステーブルからのデータに対するETL操作を実行し、StarRocks内部テーブルにデータをロードします。データソースは内部テーブルまたは外部テーブルのいずれか、またはクラウドストレージ上のデータファイルである場合があります。ターゲットテーブルは、StarRocksの内部テーブルである必要があります。詳細な手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

### 内部または外部テーブルから内部テーブルにデータを挿入

> **注**
>
> 外部テーブルからデータを挿入することは、内部テーブルからデータを挿入することと同様です。簡単のため、以下の例では内部テーブルからデータの挿入方法のみを示します。

- 次の例は、ソーステーブルのデータをターゲットテーブル`insert_wiki_edit`に挿入します。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_1
SELECT * FROM source_wiki_edit;
```

- 次の例は、ソーステーブルのデータをターゲットテーブル`insert_wiki_edit`の`p06`と`p12`パーティションに挿入します。パーティションが指定されていない場合、データはすべてのパーティションに挿入されます。それ以外の場合、データは指定されたパーティションにのみ挿入されます。

```SQL
INSERT INTO insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_2
SELECT * FROM source_wiki_edit;
```

データがそれぞれのテーブルに挿入されていることを確認するために、ターゲットテーブルのクエリを実行してください。

```Plain text```
```SQL
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント時間         | チャンネル    | ユーザー  | 匿名           | マイナー   | 新規    | ロボット  | 未巡回         | デルタ | 追加   | 削除       |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行をセット (0.00 秒)

MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
クエリ Ok、0 行が変更されました (0.01 秒)

MySQL > select * from insert_wiki_edit;
空のセット (0.00 秒)

- 次の例は、ソーステーブルの `event_time` および `channel` 列をターゲットテーブル `insert_wiki_edit` に挿入します。ここで指定されていない列にはデフォルト値が使用されます。

INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;

###ファイルを使用して外部ソースからデータを直接挿入する

v3.1以降、StarRocksは[FILES()](../sql-reference/sql-functions/table-functions/files.md)関数を使用してクラウドストレージのファイルからデータを直接ロードすることをサポートしているため、まず外部カタログまたは外部ファイルテーブルを作成する必要はありません。さらに、FILES()はファイルのテーブルスキーマを自動的に推測することができます。

次の例では、AWS S3バケット `inserttest` 内のParquetファイル **parquet/insert_wiki_edit_append.parquet** からテーブル `insert_wiki_edit` にデータ行を挿入します。

```Plain
INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
```

## INSERT OVERWRITE VALUESを使用したデータの上書き

INSERT OVERWRITE VALUESコマンドを使用して、1つまたは複数の行で特定のテーブルを上書きできます。複数の行はコンマ（、）で区切られます。詳しい手順やパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

> **注意**
>
> INSERT OVERWRITE VALUESを使用してデータを上書きするのは、小規模なデータセットを使用してデモを確認する場合にのみ適用されます。大規模なテストや本番環境にはお勧めしません。StarRocksに大量のデータをロードする場合は、シナリオに合った他のオプションについては、[インジェスション概要](../loading/Loading_intro.md)を参照してください。

ソーステーブルとターゲットテーブルをクエリして、それぞれにデータが含まれていることを確認してください。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント時間         | チャンネル    | ユーザー  | 匿名           | マイナー   | 新規    | ロボット  | 未巡回         | デルタ | 追加   | 削除       |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行をセット (0.02 秒)

MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント時間         | チャンネル    | ユーザー  | 匿名           | マイナー   | 新規    | ロボット  | 未巡回         | デルタ | 追加   | 削除       |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行をセット (0.01 秒)

次の例では、ソーステーブル `source_wiki_edit` を新しい行2つで上書きします。

```SQL
INSERT OVERWRITE source_wiki_edit
WITH LABEL insert_load_wikipedia_ow
VALUES
    ("2015-09-12 00:00:00","#cn.wikipedia","GELongstreet",0,0,0,0,0,36,36,0),
    ("2015-09-12 00:00:00","#fr.wikipedia","PereBot",0,1,0,1,0,17,17,0);
```

## INSERT OVERWRITE SELECTを使用したデータの上書き

INSERT OVERWRITE SELECTコマンドを使用して、データソーステーブルからのクエリ結果でテーブルを上書きできます。INSERT OVERWRITE SELECTステートメントは、1つ以上の内部または外部テーブルのデータに対するETL操作を実行し、データを内部テーブルに上書きます。詳しい手順やパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

> **注意**
>
> 外部テーブルからデータをロードすることは、内部テーブルからデータをロードすることと同じです。単純化のため、以下の例では内部テーブルのデータでターゲットテーブルを上書きする方法のみを示しています。

異なるデータ行を含むソーステーブルとターゲットテーブルをクエリしてください。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント時間         | チャンネル    | ユーザー     | 匿名           | マイナー   | 新規    | ロボット  | 未巡回         | デルタ | 追加   | 削除       |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot      |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行をセット (0.02 秒)

MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント時間         | チャンネル    | ユーザー  | 匿名           | マイナー   | 新規    | ロボット  | 未巡回         | デルタ | 追加   | 削除       |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行をセット (0.01 秒)
```
- 以下の例では、テーブル`insert_wiki_edit`をソーステーブルのデータで上書きします。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_1
SELECT * FROM source_wiki_edit;
```

- 以下の例では、テーブル`insert_wiki_edit`の`p06`および`p12`のパーティションをソーステーブルのデータで上書きします。

```SQL
INSERT OVERWRITE insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_ow_2
SELECT * FROM source_wiki_edit;
```

ターゲットテーブルをクエリしてデータが存在することを確認します。

```plain text
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user         | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot      |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.01 sec)
```

`p06`および`p12`パーティションを切り詰めると、クエリでデータが返されなくなります。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 以下の例では、ソーステーブルからの`event_time`および`channel`列でターゲットテーブル`insert_wiki_edit`を上書きします。データが上書きされない列にはデフォルト値が割り当てられます。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

## 生成列を持つテーブルにデータを挿入

生成列は、事前に定義された式または他の列に基づいて導出される特別な列です。生成列は、クエリが高額な式を評価する場合に特に役立ちます。例えば、JSON値から特定のフィールドをクエリしたり、ARRAYデータを計算したりする場合などです。StarRocksは、テーブルにデータが読み込まれる際に式を評価して結果を生成列に格納し、クエリ中の式の評価を回避し、クエリのパフォーマンスを向上させます。

INSERTを使用して生成列を持つテーブルにデータをロードできます。

以下の例では、テーブル`insert_generated_columns`を作成し、その中に1行挿入します。テーブルには2つの生成列（`avg_array`および`get_string`）が含まれています。`avg_array`は`data_array`内のARRAYデータの平均値を計算し、`get_string`は`data_json`のJSONパス`a`から文字列を抽出します。

```SQL
CREATE TABLE insert_generated_columns (
  id           INT(11)           NOT NULL    COMMENT "ID",
  data_array   ARRAY<INT(11)>    NOT NULL    COMMENT "ARRAY",
  data_json    JSON              NOT NULL    COMMENT "JSON",
  avg_array    DOUBLE            NULL 
      AS array_avg(data_array)               COMMENT "Get the average of ARRAY",
  get_string   VARCHAR(65533)    NULL 
      AS get_json_string(json_string(data_json), '$.a') COMMENT "Extract JSON string"
) ENGINE=OLAP 
PRIMARY KEY(id)
DISTRIBUTED BY HASH(id);

INSERT INTO insert_generated_columns 
VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
```

> **注意**
>
> 生成列にデータを直接ロードすることはサポートされていません。

テーブル内のデータを確認するためにクエリを実行できます。

```Plain
mysql> SELECT * FROM insert_generated_columns;
+------+------------+------------------+-----------+------------+
| id   | data_array | data_json        | avg_array | get_string |
+------+------------+------------------+-----------+------------+
|    1 | [1,2]      | {"a": 1, "b": 2} |       1.5 | 1          |
+------+------------+------------------+-----------+------------+
1 row in set (0.02 sec)
```

## INSERTを使用してデータを非同期でロードする

INSERTでデータをロードすることは同期的なトランザクションを送信します。セッションの中断やタイムアウトのために失敗する可能性があります。[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を使用して非同期のINSERTトランザクションを送信できます。この機能はStarRocks v2.5からサポートされています。

- 以下の例は、ソーステーブルからデータをターゲットテーブル`insert_wiki_edit`に非同期で挿入します。

```SQL
SUBMIT TASK AS INSERT INTO insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下の例は、テーブル`insert_wiki_edit`をソーステーブルのデータで非同期に上書きします。

```SQL
SUBMIT TASK AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下の例は、ヒントを使用してクエリのタイムアウトを`100000`秒に延長して、テーブル`insert_wiki_edit`をソーステーブルのデータで非同期に上書きします。

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下の例は、タスク名を`async`として指定し、テーブル`insert_wiki_edit`をソーステーブルのデータで非同期に上書きします。

```SQL
SUBMIT TASK async
AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

非同期のINSERTタスクのステータスは、Information Schemaのメタデータビュー`task_runs`をクエリして確認できます。

以下の例は非同期INSERTタスク`async`のステータスをチェックしています。

```SQL
SELECT * FROM information_schema.task_runs WHERE task_name = 'async';
```

## INSERTジョブのステータスを確認

### 結果を使用してチェック

同期的なINSERTトランザクションは、トランザクションの結果に応じて異なるステータスを返します。

- **トランザクションが成功した場合**

トランザクションが成功した場合、StarRocksは次の内容を返します。

```Plain
Query OK, 2 rows affected (0.05 sec)
{'label':'insert_load_wikipedia', 'status':'VISIBLE', 'txnId':'1006'}
```

- **トランザクションが失敗した場合**

すべてのデータ行がターゲットテーブルにロードされない場合、INSERTトランザクションは失敗します。トランザクションが失敗した場合、StarRocksは次の内容を返します。

```Plain
ERROR 1064 (HY000): Insert has filtered data in strict mode, tracking_url=http://x.x.x.x:yyyy/api/_load_error_log?file=error_log_9f0a4fd0b64e11ec_906bbede076e9d08
```

`tracking_url`を使用してログを確認して問題を特定できます。

### Information Schemaを使用してチェック

[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して、`information_schema`データベースの`loads`テーブルから1つまたは複数のロードジョブの結果をクエリできます。この機能はv3.1からサポートされています。

例1：`load_test`データベースで実行されたロードジョブの結果をクエリし、作成時間（`CREATE_TIME`）で結果を降順にソートし、トップの結果のみを返します。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test'
ORDER BY create_time DESC
LIMIT 1\G
```

例2：`load_test`データベースで実行されたロードジョブ（ラベルが`insert_load_wikipedia`）の結果をクエリします。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test' and label = 'insert_load_wikipedia'\G
```

結果は以下の通りです。

```Plain
*************************** 1. row ***************************
              JOB_ID: 21319
               LABEL: insert_load_wikipedia
       DATABASE_NAME: load_test
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: INSERT
            PRIORITY: NORMAL
           SCAN_ROWS: 0
       FILTERED_ROWS: 0
     UNSELECTED_ROWS: 0
           SINK_ROWS: 2
            ETL_INFO: 
           TASK_INFO: resource:N/A; timeout(s):300; max_filter_ratio:0.0
         CREATE_TIME: 2023-08-09 10:42:23
      ETL_START_TIME: 2023-08-09 10:42:23
     ETL_FINISH_TIME: 2023-08-09 10:42:23
     LOAD_START_TIME: 2023-08-09 10:42:23
```
    LOAD_FINISH_TIME: 2023-08-09 10:42:24
         JOB_DETAILS: {"All backends":{"5ebf11b5-365e-11ee-9e4a-7a563fb695da":[10006]},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":175,"InternalTableLoadRows":2,"ScanBytes":0,"ScanRows":0,"TaskNumber":1,"Unfinished backends":{"5ebf11b5-365e-11ee-9e4a-7a563fb695da":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: NULL
        TRACKING_SQL: NULL
REJECTED_RECORD_PATH: NULL
1 row in set (0.01 sec)
```

For information about the fields in the return results, see [Information Schema > loads](../reference/information_schema/loads.md).

### curlコマンドを使用して確認します

curlコマンドを使用して、INSERTトランザクションのステータスを確認できます。

ターミナルを起動し、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> \
  http://<fe_address>:<fe_http_port>/api/<db_name>/_load_info?label=<label_name>
```

次の例は、ラベル`insert_load_wikipedia`のトランザクションのステータスを確認しています。

```Bash
curl --location-trusted -u <username>:<password> \
  http://x.x.x.x:8030/api/load_test/_load_info?label=insert_load_wikipedia
```

> **注記**
>
> パスワードを設定していないアカウントを使用する場合は、`<username>:`のみを入力する必要があります。

結果は次のようになります。

```Plain
{
   "jobInfo":{
      "dbName":"load_test",
      "tblNames":[
         "source_wiki_edit"
      ],
      "label":"insert_load_wikipedia",
      "state":"FINISHED",
      "failMsg":"",
      "trackingUrl":""
   },
   "status":"OK",
   "msg":"Success"
}
```

## 設定

INSERTトランザクションのために次の構成項目を設定できます。

- **FE configuration**

| FE configuration                   | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| insert_load_default_timeout_second | INSERTトランザクションのデフォルトのタイムアウト設定。単位：秒。このパラメータで設定された時間内に、現在のINSERTトランザクションが完了しない場合、システムによってキャンセルされ、ステータスがCANCELLEDになります。StarRocksの現行バージョンでは、このパラメータを使用してすべてのINSERTトランザクションに対して均一なタイムアウトを指定することができますが、特定のINSERTトランザクションに対して異なるタイムアウトを設定することはできません。デフォルト値は3600秒（1時間）です。指定された時間内にINSERTトランザクションが完了しない場合、このパラメータを調整してタイムアウトを延長できます。 |

- **セッション変数**

| Session variable     | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| enable_insert_strict | INSERTトランザクションが無効なデータ行に対して寛大であるかどうかを制御するためのスイッチ値。 `true`に設定すると、データ行のいずれかが無効な場合、トランザクションは失敗します。 `false`に設定すると、少なくとも1行のデータが正しくロードされた場合にトランザクションが成功し、ラベルが返されます。デフォルト値は`true`です。この変数は`SET enable_insert_strict = {true or false};`コマンドで設定できます。 |
| query_timeout        | SQLコマンドのタイムアウト。単位：秒。SQLコマンドとしてのINSERTも、このセッション変数によって制限されます。この変数は`SET query_timeout = xxx;`コマンドで設定できます。 |
```