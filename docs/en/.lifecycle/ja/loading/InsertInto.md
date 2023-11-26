---
displayed_sidebar: "Japanese"
---

# INSERTを使用してデータをロードする

このトピックでは、SQLステートメントであるINSERTを使用してStarRocksにデータをロードする方法について説明します。

MySQLや他の多くのデータベース管理システムと同様に、StarRocksはINSERTを使用して内部テーブルにデータをロードすることができます。VALUES句を使用して1つ以上の行を直接挿入することで、関数やデモのテストに使用することができます。また、[外部テーブル](../data_source/External_table.md)から内部テーブルにクエリの結果によって定義されたデータを挿入することもできます。StarRocks v3.1以降では、INSERTコマンドとテーブル関数[FILES()](../sql-reference/sql-functions/table-functions/files.md)を使用して、クラウドストレージ上のファイルから直接データをロードすることもできます。

StarRocks v2.4では、INSERT OVERWRITEを使用してテーブルにデータを上書きすることもサポートされています。INSERT OVERWRITEステートメントは、次の操作を統合して上書き機能を実装します。

1. 元のデータを格納するパーティションに一時パーティションを作成します。
2. データを一時パーティションに挿入します。
3. 元のパーティションと一時パーティションを入れ替えます。

> **注意**
>
> パーティションを入れ替える前にデータを上書きする前にデータを検証する必要がある場合は、INSERT OVERWRITEの代わりに上記の手順に従ってデータを上書きし、パーティションを入れ替える前にデータを検証することができます。

## 注意事項

- 同期的なINSERTトランザクションをキャンセルするには、MySQLクライアントから**Ctrl**キーと**C**キーを押すだけです。
- [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を使用して非同期のINSERTタスクを送信することができます。
- StarRocksの現在のバージョンでは、テーブルのスキーマに準拠しない行のデータがある場合、デフォルトではINSERTトランザクションは失敗します。たとえば、テーブルのマッピングフィールドの長さ制限を超えるフィールドの長さがある行がある場合、INSERTトランザクションは失敗します。テーブルと一致しない行をフィルタリングしてトランザクションを続行するには、セッション変数`enable_insert_strict`を`false`に設定することができます。
- INSERTステートメントを頻繁に実行してStarRocksにデータを少量ずつロードする場合、過剰なデータバージョンが生成されます。これはクエリのパフォーマンスに重大な影響を与えます。本番環境では、INSERTコマンドを頻繁に使用してデータをロードすることはお勧めしません。ストリーミングデータや小規模なデータバッチをロードするためのソリューションが必要な場合は、データソースとしてApache Kafka®を使用し、[Routine Load](../loading/RoutineLoad.md)を使用してデータをロードすることをお勧めします。
- INSERT OVERWRITEステートメントを実行する場合、StarRocksは元のデータを格納するパーティションに一時パーティションを作成し、新しいデータを一時パーティションに挿入し、[元のパーティションと一時パーティションを入れ替えます](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#use-a-temporary-partition-to-replace-current-partition)。これらの操作はFEリーダーノードで実行されます。したがって、INSERT OVERWRITEコマンドの実行中にFEリーダーノードがクラッシュすると、ロードトランザクション全体が失敗し、一時パーティションが切り捨てられます。

## 準備

`load_test`という名前のデータベースを作成し、宛先テーブルとして`insert_wiki_edit`、ソーステーブルとして`source_wiki_edit`を作成します。

> **注意**
>
> このトピックで示される例は、`insert_wiki_edit`テーブルと`source_wiki_edit`テーブルを基にしています。独自のテーブルとデータで作業する場合は、準備をスキップして次のステップに進むことができます。

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
> StarRocks v2.5.7以降、テーブルを作成するかパーティションを追加する際に、バケットの数（BUCKETS）を自動的に設定することができます。バケットの数を手動で設定する必要はありません。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

## INSERT INTO VALUESを使用してデータを挿入する

INSERT INTO VALUESコマンドを使用して、特定のテーブルに1つ以上の行を追加することができます。複数の行はカンマ（,）で区切られます。詳しい手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

> **注意**
>
> INSERT INTO VALUESを使用してデータを挿入するのは、デモの小さなデータセットでの検証が必要な場合にのみ適用されます。大量のテストや本番環境ではお勧めしません。大量のデータをStarRocksにロードするには、[Ingestion Overview](../loading/Loading_intro.md)を参照して、シナリオに合った他のオプションを使用してください。

次の例では、ラベル`insert_load_wikipedia`を持つデータソーステーブル`source_wiki_edit`に2つの行を挿入します。ラベルはデータロードトランザクションごとの一意の識別ラベルです。

```SQL
INSERT INTO source_wiki_edit
WITH LABEL insert_load_wikipedia
VALUES
    ("2015-09-12 00:00:00","#en.wikipedia","AustinFF",0,0,0,0,0,21,5,0),
    ("2015-09-12 00:00:00","#ca.wikipedia","helloSR",0,1,0,1,0,3,23,0);
```

## INSERT INTO SELECTを使用してデータを挿入する

INSERT INTO SELECTコマンドを使用して、データソーステーブルのクエリ結果をターゲットテーブルにロードすることができます。INSERT INTO SELECTコマンドは、データソーステーブルのデータに対してETL操作を実行し、StarRocksの内部テーブルにデータをロードします。データソースは、1つ以上の内部または外部テーブル、またはクラウドストレージ上のデータファイルであることができます。ターゲットテーブルは、StarRocksの内部テーブルである必要があります。詳しい手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

### 内部または外部テーブルから内部テーブルにデータを挿入する

> **注意**
>
> 外部テーブルからデータを挿入する場合は、内部テーブルからデータを挿入するのと同じです。簡単にするために、以下の例では内部テーブルからデータを挿入する方法のみを示します。

- 次の例では、ソーステーブルのデータをターゲットテーブル`insert_wiki_edit`に挿入します。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_1
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータをターゲットテーブル`insert_wiki_edit`の`p06`および`p12`パーティションに挿入します。パーティションが指定されていない場合、データはすべてのパーティションに挿入されます。それ以外の場合、データは指定されたパーティションのみに挿入されます。

```SQL
INSERT INTO insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_2
SELECT * FROM source_wiki_edit;
```

ターゲットテーブルにデータがあることを確認するために、クエリを実行します。

```Plain text
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.00 sec)
```

`p06`および`p12`パーティションを切り捨てると、クエリでデータが返されなくなります。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 次の例では、ソーステーブルの`event_time`と`channel`の列をターゲットテーブル`insert_wiki_edit`に挿入します。ここで、指定されていない列にはデフォルト値が使用されます。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

### FILES()を使用して外部ソースのファイルからデータを直接挿入する

v3.1以降、StarRocksはINSERTコマンドと[FILES()](../sql-reference/sql-functions/table-functions/files.md)関数を使用して、クラウドストレージ上のファイルから直接データをロードすることができます。そのため、最初に外部カタログや外部ファイルテーブルを作成する必要はありません。さらに、FILES()はファイルのテーブルスキーマを自動的に推測するため、データロードのプロセスを大幅に簡素化します。

次の例では、AWS S3バケット`inserttest`内のParquetファイル**parquet/insert_wiki_edit_append.parquet**からデータ行をテーブル`insert_wiki_edit`に挿入します。

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

## INSERT OVERWRITE VALUESを使用してデータを上書きする

INSERT OVERWRITE VALUESコマンドを使用して、特定のテーブルを1つ以上の行で上書きすることができます。複数の行はカンマ（,）で区切られます。詳しい手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

> **注意**
>
> INSERT OVERWRITE VALUESを使用してデータを上書きするのは、デモの小さなデータセットでの検証が必要な場合にのみ適用されます。大量のテストや本番環境ではお勧めしません。大量のデータをStarRocksにロードするには、[Ingestion Overview](../loading/Loading_intro.md)を参照して、シナリオに合った他のオプションを使用してください。

ソーステーブルとターゲットテーブルのデータをクエリして、異なる行のデータが含まれていることを確認します。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.02 sec)
 
MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.01 sec)
```

次の例では、ソーステーブルのデータでソーステーブルを上書きします。

```SQL
INSERT OVERWRITE source_wiki_edit
WITH LABEL insert_load_wikipedia_ow
VALUES
    ("2015-09-12 00:00:00","#cn.wikipedia","GELongstreet",0,0,0,0,0,36,36,0),
    ("2015-09-12 00:00:00","#fr.wikipedia","PereBot",0,1,0,1,0,17,17,0);
```

## INSERT OVERWRITE SELECTを使用してデータを上書きする

INSERT OVERWRITE SELECTコマンドを使用して、データソーステーブルのクエリ結果でテーブルを上書きすることができます。INSERT OVERWRITE SELECTステートメントは、データソーステーブルのデータに対してETL操作を実行し、データを内部テーブルに上書きします。データソースは、1つ以上の内部または外部テーブル、またはクラウドストレージ上のデータファイルであることができます。詳しい手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

> **注意**
>
> 外部テーブルからデータをロードする場合は、内部テーブルからデータをロードするのと同じです。簡単にするために、以下の例では内部テーブルからデータをロードする方法のみを示します。

ソーステーブルとターゲットテーブルのデータをクエリして、異なる行のデータが含まれていることを確認します。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user         | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot      |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.02 sec)
 
MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.01 sec)
```

- 次の例では、ターゲットテーブル`insert_wiki_edit`をソーステーブルのデータで上書きします。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_1
SELECT * FROM source_wiki_edit;
```

- 次の例では、ターゲットテーブル`insert_wiki_edit`の`p06`および`p12`パーティションをソーステーブルのデータで上書きします。

```SQL
INSERT OVERWRITE insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_ow_2
SELECT * FROM source_wiki_edit;
```

ターゲットテーブルにデータがあることを確認するために、クエリを実行します。

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

- 次の例では、ターゲットテーブル`insert_wiki_edit`をソーステーブルの`event_time`と`channel`の列で上書きします。データが上書きされない列にはデフォルト値が割り当てられます。

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

生成列は、事前に定義された式または他の列に基づいて値が導出される特殊な列です。生成列は、クエリ要求が高価な式の評価（たとえば、JSON値から特定のフィールドをクエリするか、ARRAYデータを計算するなど）を含む場合に特に有用です。StarRocksは、データがテーブルにロードされる際に式を評価し、結果を生成列に格納するため、クエリ時の式評価を回避し、クエリのパフォーマンスを向上させることができます。

INSERTを使用して生成列を持つテーブルにデータをロードすることができます。

次の例では、テーブル`insert_generated_columns`を作成し、1行を挿入します。このテーブルには2つの生成列、`avg_array`と`get_string`が含まれています。`avg_array`は`data_array`のARRAYデータの平均値を計算し、`get_string`は`data_json`のJSONパス`a`から文字列を抽出します。

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

テーブル内のデータを確認するために、クエリを実行します。

```Plain
mysql> SELECT * FROM insert_generated_columns;
+------+------------+------------------+-----------+------------+
| id   | data_array | data_json        | avg_array | get_string |
+------+------------+------------------+-----------+------------+
|    1 | [1,2]      | {"a": 1, "b": 2} |       1.5 | 1          |
+------+------------+------------------+-----------+------------+
1 row in set (0.02 sec)
```

## INSERTを使用して非同期にデータをロードする

INSERTを使用してデータをロードすると、同期トランザクションが送信され、セッションの中断やタイムアウトにより失敗する場合があります。[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を使用して非同期のINSERTトランザクションを送信することができます。この機能はStarRocks v2.5以降でサポートされています。

- 次の例では、ソーステーブルのデータをターゲットテーブル`insert_wiki_edit`に非同期に挿入します。

```SQL
SUBMIT TASK AS INSERT INTO insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータでターゲットテーブルを非同期に上書きします。

```SQL
SUBMIT TASK AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータでターゲットテーブルを非同期に上書きし、ヒントを使用してクエリのタイムアウトを`100000`秒に延長します。

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータでターゲットテーブルを非同期に上書きし、タスク名を`async`と指定します。

```SQL
SUBMIT TASK async
AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

非同期INSERTタスクのステータスは、Information Schemaのメタデータビュー`task_runs`をクエリすることで確認することができます。

次の例では、INSERTタスク`async`のステータスを確認します。

```SQL
SELECT * FROM information_schema.task_runs WHERE task_name = 'async';
```

## INSERTジョブのステータスを確認する

### 結果を使用して確認する

同期的なINSERTトランザクションは、トランザクションの結果に応じて異なるステータスを返します。

- **トランザクションが成功した場合**

トランザクションが成功した場合、StarRocksは次のような結果を返します。

```Plain
Query OK, 2 rows affected (0.05 sec)
{'label':'insert_load_wikipedia', 'status':'VISIBLE', 'txnId':'1006'}
```

- **トランザクションが失敗した場合**

すべてのデータ行がターゲットテーブルにロードされなかった場合、INSERTトランザクションは失敗します。トランザクションが失敗した場合、StarRocksは次のような結果を返します。

```Plain
ERROR 1064 (HY000): Insert has filtered data in strict mode, tracking_url=http://x.x.x.x:yyyy/api/_load_error_log?file=error_log_9f0a4fd0b64e11ec_906bbede076e9d08
```

`tracking_url`を使用してログを確認することで、問題の位置を特定することができます。

### Information Schemaを使用して確認する

[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して、`information_schema`データベースの`loads`テーブルから1つ以上のロードジョブの結果をクエリすることができます。この機能はv3.1以降でサポートされています。

例1：`load_test`データベースで実行されたロードジョブの結果をクエリし、作成時間（`CREATE_TIME`）で降順にソートし、最上位の結果のみを返す。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test'
ORDER BY create_time DESC
LIMIT 1\G
```

例2：`load_test`データベースで実行されたロードジョブ（ラベルが`insert_load_wikipedia`であるもの）の結果をクエリします。

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
    LOAD_FINISH_TIME: 2023-08-09 10:42:24
         JOB_DETAILS: {"All backends":{"5ebf11b5-365e-11ee-9e4a-7a563fb695da":[10006]},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":175,"InternalTableLoadRows":2,"ScanBytes":0,"ScanRows":0,"TaskNumber":1,"Unfinished backends":{"5ebf11b5-365e-11ee-9e4a-7a563fb695da":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: NULL
        TRACKING_SQL: NULL
REJECTED_RECORD_PATH: NULL
1 row in set (0.01 sec)
```

戻り値のフィールドについての詳細は、[Information Schema > loads](../reference/information_schema/loads.md)を参照してください。

### curlコマンドを使用して確認する

curlコマンドを使用してINSERTトランザクションのステータスを確認することができます。

ターミナルを起動し、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> \
  http://<fe_address>:<fe_http_port>/api/<db_name>/_load_info?label=<label_name>
```

次の例では、ラベルが`insert_load_wikipedia`のトランザクションのステータスを確認します。

```Bash
curl --location-trusted -u <username>:<password> \
  http://x.x.x.x:8030/api/load_test/_load_info?label=insert_load_wikipedia
```

> **注意**
>
> パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力する必要があります。

結果は次のようになります。

```Plain
{
   "jobInfo":{
      "dbName":"load_test",
      "tblNames":[
         "source_wiki_edit"
      ],
```


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

INSERTトランザクションのために、以下の設定項目を設定することができます：

- **FE設定**

| FE設定                           | 説明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| insert_load_default_timeout_second | INSERTトランザクションのデフォルトタイムアウト。単位：秒。現在のINSERTトランザクションがこのパラメータで設定された時間内に完了しない場合、システムによってキャンセルされ、ステータスがCANCELLEDになります。StarRocksの現在のバージョンでは、このパラメータを使用してすべてのINSERTトランザクションに一様なタイムアウトを指定することしかできず、特定のINSERTトランザクションに異なるタイムアウトを設定することはできません。デフォルトは3600秒（1時間）です。指定された時間内にINSERTトランザクションを完了できない場合は、このパラメータを調整してタイムアウトを延長することができます。 |

- **セッション変数**

| セッション変数       | 説明                                                         |
| -------------------- | ------------------------------------------------------------ |
| enable_insert_strict | INSERTトランザクションが無効なデータ行に対して許容的かどうかを制御するスイッチの値。`true`に設定されている場合、データ行のいずれかが無効な場合、トランザクションは失敗します。`false`に設定されている場合、少なくとも1行のデータが正しくロードされた場合にトランザクションは成功し、ラベルが返されます。デフォルトは`true`です。この変数は`SET enable_insert_strict = {trueまたはfalse};`コマンドで設定することができます。 |
| query_timeout        | SQLコマンドのタイムアウト。単位：秒。INSERTもSQLコマンドとしてこのセッション変数に制限されます。`SET query_timeout = xxx;`コマンドでこの変数を設定することができます。 |
