```yaml
---
displayed_sidebar: "Japanese"
---

# INSERTを使用してデータをロードする

../assets/commonMarkdown/insertPrivNote.mdからInsertPrivNoteをインポートします。

このトピックでは、SQLステートメントであるINSERTを使用してStarRocksにデータをロードする方法について説明します。

StarRocksは、MySQLや他の多くのデータベース管理システムと同様に、INSERTを使用して内部テーブルにデータをロードすることをサポートしています。VALUES句を使用して1つ以上の行を直接挿入して、関数やDEMOをテストすることができます。また、クエリの結果によって定義されたデータを[外部テーブル](../data_source/External_table.md)から内部テーブルに挿入することもできます。StarRocks v3.1以降、INSERTコマンドとテーブル関数[FILES()](../sql-reference/sql-functions/table-functions/files.md)を使用してクラウドストレージ上のファイルから直接データをロードすることもできます。

StarRocks v2.4では、INSERT OVERWRITEを使用してテーブルにデータを上書きすることもサポートされています。INSERT OVERWRITEステートメントには、以下の操作が統合されて上書き機能が実装されています。

1. オリジナルデータを保存するパーティションに基づいて一時パーティションを作成します。
2. データを一時パーティションに挿入します。
3. オリジナルのパーティションと一時的なパーティションを入れ替えます。

> **注意**
>
> データを上書きする前にデータを検証する必要がある場合は、INSERT OVERWRITEを使用せずに、パーティションを入れ替える前にデータを上書きし、検証してください。

## 注意事項

- 同期的なINSERTトランザクションをキャンセルするには、MySQLクライアントから**Ctrl**キーと**C**キーを押すことしかできません。
- 非同期のINSERTタスクを提出するには、[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を使用してください。
- 現在のStarRocksのバージョンでは、任意の行のデータがテーブルのスキーマに準拠していない場合、INSERTトランザクションはデフォルトで失敗します。たとえば、テーブル内のマッピングフィールドの長さを超える行がある場合、INSERTトランザクションは失敗します。テーブルと一致しない行をフィルタリングしてトランザクションを続行するには、セッション変数`enable_insert_strict`を`false`に設定できます。
- StarRocksに小さなデータバッチを頻繁にロードするためにINSERTステートメントを実行すると、大量のデータバージョンが生成されます。これはクエリのパフォーマンスに深刻な影響を与えます。本番環境では、INSERTコマンドを頻繁に使用したり、データのローディングを日常的なルーチンとして使用することはお勧めしません。アプリケーションや解析シナリオがストリーミングデータのローディングや小さなデータバッチを別々に要求する場合、Apache Kafka®をデータソースとして使用して、[Routine Load](../loading/RoutineLoad.md)を介してデータをロードすることをお勧めします。
- INSERT OVERWRITEステートメントを実行すると、StarRocksはオリジナルのデータを保存するパーティションに一時的なパーティションを作成し、新しいデータを一時的なパーティションに挿入し、[オリジナルのパーティションを一時的なパーティションで入れ替えます](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#use-a-temporary-partition-to-replace-current-partition)。これらの操作はすべてFEリーダーノードで実行されます。そのため、INSERT OVERWRITEコマンドを実行中にFEリーダーノードがクラッシュすると、ロードトランザクション全体が失敗し、一時的なパーティションが切り捨てられます。

## 準備

### 権限をチェックする

<InsertPrivNote />の権限をチェックします。

### オブジェクトを作成する

`load_test`という名前のデータベースを作成し、宛先テーブルとして`insert_wiki_edit`とソーステーブルとして`source_wiki_edit`を作成します。

> **注意**
>
> 本トピックで示される例は、テーブル`insert_wiki_edit`とテーブル`source_wiki_edit`をベースにしています。独自のテーブルとデータで作業する場合は、準備をスキップして次のステップに進んでください。

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

> **お知らせ**
>
> v2.5.7以降、StarRocksはテーブルを作成したりパーティションを追加する際に、バケツの数（BUCKETS）を自動的に設定できます。バケツの数を手動で設定する必要はもはやありません。詳細については、[バケツの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

## INSERT INTO VALUESを使用してデータを挿入する

INSERT INTO VALUESコマンドを使用して、特定のテーブルに1つ以上の行を追加できます。複数行を挿入する場合は、カンマ（,）で区切ります。詳しい手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

> **注意**
>
> INSERT INTO VALUESを使用してデータを挿入するのは、小さなデータセットでDEMOを検証する場合にのみ適用されます。大規模なテストや本番環境にはお勧めしません。StarRocksに大量のデータをロードするには、シナリオに応じた他のオプションである[Ingestion Overview](../loading/Loading_intro.md)を参照してください。

次の例では、「insert_load_wikipedia」というラベルでデータソーステーブル`source_wiki_edit`に2行を挿入します。ラベルはデータロードトランザクションごとに一意の識別ラベルです。

```SQL
INSERT INTO source_wiki_edit
WITH LABEL insert_load_wikipedia
VALUES
    ("2015-09-12 00:00:00","#en.wikipedia","AustinFF",0,0,0,0,0,21,5,0),
    ("2015-09-12 00:00:00","#ca.wikipedia","helloSR",0,1,0,1,0,3,23,0);
```

## INSERT INTO SELECTを使用してデータを挿入する

INSERT INTO SELECTコマンドを使用して、データソーステーブル上のクエリの結果を対象テーブルにロードできます。INSERT INTO SELECTコマンドは、データソーステーブルのデータに対してETL操作を実行し、データをStarRocksの内部テーブルにロードします。データソースには、1つ以上の内部または外部テーブル、あるいはクラウドストレージ上のデータファイルが利用できます。対象テーブルはStarRocksの内部テーブルである必要があります。詳しい手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

### 内部または外部テーブルから内部テーブルにデータを挿入する

> **注意**
>
> 外部テーブルからデータを挿入する場合、内部テーブルからデータを挿入するのと同じです。わかりやすくするために、以下の例では内部テーブルからデータを挿入する方法のみを示します。

- 次の例では、ソーステーブルのデータを対象テーブル`insert_wiki_edit`に挿入します。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_1
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータを対象テーブル`insert_wiki_edit`の`p06`および`p12`パーティションに挿入します。パーティションが指定されていない場合は、すべてのパーティションにデータが挿入されます。そうでない場合は、指定されたパーティションにのみデータが挿入されます。

```SQL
INSERT INTO insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_2
SELECT * FROM source_wiki_edit;
```

対象テーブルのクエリを実行して、データが存在することを確認します。

```plainttext
```SQL
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント_時刻        | チャンネル     | ユーザ     | 匿名         | マイナー   | 新規   | ロボット  | 未巡回       | デルタ  | 追加   | 削除     |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行がセット (0.00 秒)
```

- "p06" および "p12" パーティションを切断すると、クエリにデータが返されなくなります。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
クエリは正常に完了しました。0 行が影響を受けました (0.01 秒)

MySQL > select * from insert_wiki_edit;
空のセット (0.00 秒)
```

- 次の例では、ソーステーブルの "event_time" および "channel" の列をターゲットテーブル "insert_wiki_edit" に挿入します。ここで指定されていない列ではデフォルト値が使用されます。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

### FILES() を使用して外部ソースのファイルからデータを直接挿入する

v3.1 以降、StarRocks は INSERT コマンドと [FILES()](../sql-reference/sql-functions/table-functions/files.md) 関数を使用して、クラウドストレージ上のファイルからデータを直接ロードする機能をサポートしており、外部カタログやファイル外部テーブルを事前に作成する必要はありません。また、FILES() を使用するとファイルのテーブルスキーマを自動的に推測することができるため、データの読み込みプロセスが大幅に簡素化されます。

次の例では、AWS S3 バケット "inserttest" 内の Parquet ファイル **parquet/insert_wiki_edit_append.parquet** からテーブル "insert_wiki_edit" にデータの行を挿入します。

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

## INSERT OVERWRITE VALUES を使用してデータを上書きする

INSERT OVERWRITE VALUES コマンドを使用して、1 つ以上の行で特定のテーブルを上書きできます。複数の行はコンマ (,) で区切られます。詳しい手順やパラメータの参照については、[SQL Reference - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。

> **注意**
>
> INSERT OVERWRITE VALUES を使用してデータを上書きするのは、デモ確認と小規模のデータセットでのみ推奨されます。大量のテストや本番環境ではお勧めしません。大量データを StarRocks に読み込むには、シナリオに適したその他のオプションについては [Ingestion Overview](../loading/Loading_intro.md) を参照してください。

ソーステーブルとターゲットテーブルをクエリして、それぞれにデータがあることを確認します。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント_時刻          | チャンネル       | ユーザ     | 匿名         | マイナー   | 新規   | ロボット   | 未巡回           | デルタ   | 追加     | 削除      |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行がセット (0.02 秒)
 
MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント_時刻          | チャンネル       | ユーザ     | 匿名         | マイナー   | 新規   | ロボット  | 未巡回          | デルタ  | 追加   | 削除     |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行がセット (0.01 秒)
```

次の例では、ソーステーブル "source_wiki_edit" を新しい 2 行で上書きします。

```SQL
INSERT OVERWRITE source_wiki_edit
WITH LABEL insert_load_wikipedia_ow
VALUES
    ("2015-09-12 00:00:00","#cn.wikipedia","GELongstreet",0,0,0,0,0,36,36,0),
    ("2015-09-12 00:00:00","#fr.wikipedia","PereBot",0,1,0,1,0,17,17,0);
```

## INSERT OVERWRITE SELECT を使用してデータを上書きする

INSERT OVERWRITE SELECT コマンドを使用して、データソーステーブルのクエリ結果でテーブルを上書きできます。INSERT OVERWRITE SELECT ステートメントは、1 つ以上の内部または外部テーブルのデータに対して ETL 操作を実行し、内部テーブルをデータで上書きします。詳しい手順やパラメータの参照については、[SQL Reference - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。

> **注意**
>
> 外部テーブルからデータを読み込む手順は、内部テーブルからデータを読み込む手順と同じです。簡単のため、以下の例では内部テーブルのデータでターゲットテーブルを上書きする手順のみを示します。

ソーステーブルとターゲットテーブルをクエリして、それぞれに異なる行のデータがあることを確認します。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント_時刻          | チャンネル       | ユーザ               | 匿名         | マイナー  | 新規    | ロボット  | 未巡回             | デルタ   | 追加      | 削除      |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot      |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行がセット (0.02 秒)
 
MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| イベント_時刻          | チャンネル       | ユーザ     | 匿名         | マイナー   | 新規   | ロボット  | 未巡回         | デルタ  | 追加   | 削除     |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 行がセット (0.01 秒)
```

- 以下の例は、テーブル `insert_wiki_edit` のデータをソーステーブルから上書きします。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_1
SELECT * FROM source_wiki_edit;
```

- 以下の例は、テーブル `insert_wiki_edit` の`p06`および`p12` パーティションを、ソーステーブルのデータで上書きします。

```SQL
INSERT OVERWRITE insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_ow_2
SELECT * FROM source_wiki_edit;
```

ターゲットテーブルをクエリして、データが含まれていることを確認してください。

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

`p06`および`p12`パーティションを切り捨てると、クエリでデータが返されなくなります。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 以下の例は、ターゲットテーブル `insert_wiki_edit` を、ソーステーブルの`event_time`および`channel`列で上書きします。データが上書きされない列には、デフォルト値が割り当てられます。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

## 生成列を持つテーブルにデータを挿入する

生成列とは、定義済みの式または他の列に基づいて派生した値を持つ特別な列です。生成列は、例えばJSON値から特定のフィールドをクエリしたり、ARRAYデータを計算したりするようなクエリ要求が含まれる場合に特に便利です。StarRocksは式を評価して生成列に結果を格納するため、クエリ中に式評価を回避し、クエリのパフォーマンスを向上させることができます。

INSERTを使用してテーブルに生成列を持つデータをロードできます。

以下の例では、テーブル `insert_generated_columns` を作成し、そのテーブルに行を挿入します。テーブルには、`avg_array`および`get_string`という2つの生成列が含まれています。`avg_array`は`data_array`のARRAYデータの平均値を計算し、`get_string`は`data_json`のJSONパス`a`から文字列を抽出します。

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

テーブルのデータを確認するためにクエリを実行できます。

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

INSERTを使用してデータをロードすると、同期トランザクションが送信され、セッションの中断やタイムアウトのために失敗する可能性があります。StarRocks v2.5以降、[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md) を使用して非同期のINSERTトランザクションを送信できます。

- 以下の例は、ソーステーブルからターゲットテーブル `insert_wiki_edit` にデータを非同期で挿入します。

```SQL
SUBMIT TASK AS INSERT INTO insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下の例は、テーブル `insert_wiki_edit` を、ソーステーブルのデータで非同期に上書きします。

```SQL
SUBMIT TASK AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下の例は、テーブル `insert_wiki_edit` を、ソーステーブルのデータで上書きし、ヒントを使用してクエリのタイムアウトを`100000`秒に延長します。

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 以下の例は、テーブル `insert_wiki_edit` を、ソーステーブルのデータで上書きし、タスク名を`async`と指定します。

```SQL
SUBMIT TASK async
AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

非同期のINSERTタスクの状態は、情報スキーマのメタデータビュー `task_runs`をクエリして確認できます。

以下の例は、非同期のINSERTタスク `async` の状態を確認するものです。

```SQL
SELECT * FROM information_schema.task_runs WHERE task_name = 'async';
```

## INSERTジョブの状態を確認する

### 結果を使って確認

同期のINSERTトランザクションは、トランザクションの結果に応じて異なる状態を返します。

- **トランザクションが成功した場合**

トランザクションが成功した場合、StarRocksは以下を返します。

```Plain
Query OK, 2 rows affected (0.05 sec)
{'label':'insert_load_wikipedia', 'status':'VISIBLE', 'txnId':'1006'}
```

- **トランザクションが失敗した場合**

すべてのデータ行がターゲットテーブルにロードできなかった場合、INSERTトランザクションは失敗します。トランザクションが失敗した場合、StarRocksは以下を返します。

```Plain
ERROR 1064 (HY000): Insert has filtered data in strict mode, tracking_url=http://x.x.x.x:yyyy/api/_load_error_log?file=error_log_9f0a4fd0b64e11ec_906bbede076e9d08
```

`tracking_url`でログを確認して問題を特定できます。

### Information Schemaを使用して確認

[v3.1](https://github.com/starrocks/starrocks/commit/470c3705)以降のバージョンで、`information_schema`データベースの`loads`テーブルから1つまたは複数のロードジョブの結果をクエリするために[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用できます。

例1：`load_test`データベースで実行されたロードジョブの結果をクエリし、作成時間（`CREATE_TIME`）で結果を降順に並べ替えて、トップの結果のみを返します。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test'
ORDER BY create_time DESC
LIMIT 1\G
```

例2：ラベルが`insert_load_wikipedia`のロードジョブ（`load_test`データベースで実行）の結果をクエリします。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test' and label = 'insert_load_wikipedia'\G
```

結果は次のようになります。

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

結果のフィールドに関する詳細は、[Information Schema > loads](../reference/information_schema/loads.md) を参照してください。

### curlコマンドを使用して確認する

curlコマンドを使用してINSERTトランザクションのステータスを確認できます。

ターミナルを開き、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> \
  http://<fe_address>:<fe_http_port>/api/<db_name>/_load_info?label=<label_name>
```

次の例では、ラベル`insert_load_wikipedia`のトランザクションのステータスをチェックしています。

```Bash
curl --location-trusted -u <username>:<password> \
  http://x.x.x.x:8030/api/load_test/_load_info?label=insert_load_wikipedia
```

> **注意**
>
> パスワードが設定されていないアカウントを使用する場合、`<username>:`のみを入力する必要があります。

戻り値は次のとおりです。

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

INSERTトランザクションに対して次の構成項目を設定できます。

- **FE構成**

| FE構成                           | 説明                                                       |
| ---------------------------------- | ------------------------------------------------------------ |
| insert_load_default_timeout_second | INSERTトランザクションのデフォルトタイムアウト。単位：秒。現在のINSERTトランザクションがこのパラメータで設定された時間内に完了しない場合、システムによってキャンセルされ、ステータスはCANCELLEDになります。StarRocksの現行バージョンでは、このパラメータを使用してすべてのINSERTトランザクションに一様なタイムアウトを指定することしかできず、特定のINSERTトランザクションに異なるタイムアウトを設定することはできません。デフォルトは3600秒（1時間）です。指定された時間内にINSERTトランザクションを完了できない場合は、このパラメータを調整してタイムアウトを延長することができます。 |

- **セッション変数**

| セッション変数     | 説明                                                  |
| -------------------- | ------------------------------------------------------------ |
| enable_insert_strict | INSERTトランザクションが不正なデータ行に寛容かどうかを制御するためのスイッチ値。`true`に設定すると、データ行のいずれかが無効な場合、トランザクションは失敗します。`false`に設定すると、少なくとも1行のデータが正しくロードされていればトランザクションは成功し、ラベルが返されます。デフォルトは`true`です。この変数は、`SET enable_insert_strict = {true or false};`コマンドで設定できます。 |
| query_timeout        | SQLコマンドのタイムアウト。単位：秒。SQLコマンドであるINSERTも、このセッション変数で制限されます。この変数は、`SET query_timeout = xxx;`コマンドで設定できます。 |