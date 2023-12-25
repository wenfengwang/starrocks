---
displayed_sidebar: English
---

# INSERTを使用したデータのロード

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

このトピックでは、SQLステートメント - INSERTを使用してStarRocksにデータをロードする方法について説明します。

MySQLや他の多くのデータベース管理システムと同様に、StarRocksはINSERTを使用して内部テーブルにデータをロードすることをサポートしています。VALUES句を使用して1つ以上の行を直接挿入し、機能やDEMOをテストすることができます。また、[外部テーブル](../data_source/External_table.md)からのクエリ結果によって定義されたデータを内部テーブルに挿入することもできます。StarRocks v3.1以降では、INSERTコマンドとテーブル関数[FILES()](../sql-reference/sql-functions/table-functions/files.md)を使用して、クラウドストレージ上のファイルから直接データをロードすることができます。

StarRocks v2.4では、INSERT OVERWRITEを使用してテーブルにデータを上書きする機能がさらにサポートされています。INSERT OVERWRITEステートメントは、上書き機能を実装するために以下の操作を統合します：

1. 元のデータを格納しているパーティションに従って一時パーティションを作成します。
2. 一時パーティションにデータを挿入します。
3. 元のパーティションと一時パーティションを交換します。

> **注記**
>
> データを上書きする前に検証が必要な場合、INSERT OVERWRITEを使用する代わりに、上記の手順に従ってデータを上書きし、パーティションを交換する前に検証することができます。

## 注意事項

- 同期INSERTトランザクションは、MySQLクライアントから**Ctrl**キーと**C**キーを押すことでのみキャンセルできます。
- [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を使用して非同期INSERTタスクを送信できます。
- StarRocksの現在のバージョンでは、行のデータがテーブルのスキーマに準拠していない場合、INSERTトランザクションはデフォルトで失敗します。たとえば、任意の行のフィールドの長さがテーブルのマッピングフィールドの長さ制限を超える場合、INSERTトランザクションは失敗します。セッション変数`enable_insert_strict`を`false`に設定することで、テーブルと一致しない行を除外してトランザクションを続行することができます。
- INSERTステートメントを頻繁に実行して小規模なデータバッチをStarRocksにロードすると、過剰なデータバージョンが生成され、クエリのパフォーマンスが大きく影響を受けます。本番環境では、INSERTコマンドを頻繁に使用してデータをロードすることや、日常的なデータローディングルーチンとして使用することは推奨されません。アプリケーションや分析シナリオがストリーミングデータや小規模なデータバッチのローディングソリューションを別々に要求する場合は、Apache Kafka®をデータソースとして使用し、[Routine Load](../loading/RoutineLoad.md)を介してデータをロードすることをお勧めします。
- INSERT OVERWRITEステートメントを実行する場合、StarRocksは元のデータを格納するパーティションのための一時パーティションを作成し、新しいデータを一時パーティションに挿入し、[元のパーティションと一時パーティションを交換します](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#use-a-temporary-partition-to-replace-current-partition)。これらの操作はすべてFEリーダーノードで実行されるため、INSERT OVERWRITEコマンドを実行中にFEリーダーノードがクラッシュすると、ロードトランザクション全体が失敗し、一時パーティションが削除されます。

## 準備

### 権限の確認

<InsertPrivNote />

### オブジェクトの作成

`load_test`という名前のデータベースを作成し、変換先テーブルとして`insert_wiki_edit`テーブル、ソーステーブルとして`source_wiki_edit`テーブルを作成します。

> **注記**
>
> このトピックで示される例は、`insert_wiki_edit`テーブルと`source_wiki_edit`テーブルに基づいています。独自のテーブルとデータで作業を希望する場合は、準備をスキップして次のステップに進むことができます。

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
```

> **通知**
>
> v2.5.7以降、StarRocksはテーブルを作成する際やパーティションを追加する際にバケット数（BUCKETS）を自動的に設定することができます。バケット数を手動で設定する必要はもうありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

## INSERT INTO VALUESを使用したデータの挿入

INSERT INTO VALUESコマンドを使用して、特定のテーブルに1行または複数行を追加できます。複数の行はカンマ（,）で区切られます。詳細な手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

> **警告**
>
> INSERT INTO VALUESを使用したデータの挿入は、小規模なデータセットでDEMOを検証する必要がある場合にのみ適しています。大規模なテスト環境や本番環境では推奨されません。StarRocksに大量のデータをロードするには、[データ取り込みの概要](../loading/Loading_intro.md)を参照して、シナリオに適した他のオプションを検討してください。

次の例では、`source_wiki_edit`データソーステーブルに`insert_load_wikipedia`というラベルで2行を挿入します。ラベルは、データベース内の各データロードトランザクションに対する一意の識別ラベルです。

```SQL
INSERT INTO source_wiki_edit
WITH LABEL insert_load_wikipedia
VALUES
    ("2015-09-12 00:00:00","#en.wikipedia","AustinFF",0,0,0,0,0,21,5,0),
    ("2015-09-12 00:00:00","#ca.wikipedia","helloSR",0,1,0,1,0,3,23,0);
```

## INSERT INTO SELECTを使用したデータの挿入

データソーステーブルに対するクエリの結果をINSERT INTO SELECTコマンドを使用してターゲットテーブルにロードすることができます。INSERT INTO SELECTコマンドは、データソーステーブルからのデータにETL操作を実行し、StarRocksの内部テーブルにデータをロードします。データソースは、1つ以上の内部テーブルまたは外部テーブル、あるいはクラウドストレージ上のデータファイルである可能性があります。ターゲットテーブルはStarRocksの内部テーブルでなければなりません。詳細な手順とパラメータの参照については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

### 内部テーブルまたは外部テーブルから内部テーブルへのデータの挿入

> **注記**
>
> 外部テーブルからのデータの挿入は、内部テーブルからのデータの挿入と同様です。簡単のため、以下の例では内部テーブルからのデータの挿入方法のみを示します。

- 次の例では、ソーステーブルからターゲットテーブル`insert_wiki_edit`へのデータを挿入します。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_1
SELECT * FROM source_wiki_edit;
```


- 次の例では、ソーステーブルからターゲットテーブル `insert_wiki_edit` のパーティション `p06` と `p12` にデータを挿入します。パーティションが指定されていない場合、データはすべてのパーティションに挿入されます。それ以外の場合、データは指定されたパーティションにのみ挿入されます。

```SQL
INSERT INTO insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_2
SELECT * FROM source_wiki_edit;
```

ターゲットテーブルをクエリして、データが存在することを確認します。

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

`p06` と `p12` パーティションを切り捨てると、クエリでデータは返されません。

```Plain
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 次の例では、ソーステーブルからターゲットテーブル `insert_wiki_edit` に `event_time` と `channel` の列を挿入します。ここで指定されていない列にはデフォルト値が使用されます。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

### FILES() を使用して外部ソースのファイルから直接データを挿入する

v3.1 以降、StarRocks は INSERT コマンドと [FILES()](../sql-reference/sql-functions/table-functions/files.md) 関数を使用してクラウドストレージ上のファイルからデータを直接ロードすることをサポートしています。これにより、外部カタログやファイル外部テーブルを最初に作成する必要がなくなります。さらに、FILES() はファイルのテーブルスキーマを自動的に推測できるため、データロードのプロセスが大幅に簡素化されます。

次の例では、AWS S3 バケット `inserttest` 内の Parquet ファイル **parquet/insert_wiki_edit_append.parquet** からのデータ行をテーブル `insert_wiki_edit` に挿入します：

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

INSERT OVERWRITE VALUES コマンドを使用して、特定のテーブルを1つ以上の行で上書きすることができます。複数の行はコンマ (,) で区切られます。詳細な手順とパラメーターの参照については、[SQL リファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。

> **注意**
>
> INSERT OVERWRITE VALUES によるデータの上書きは、小規模なデータセットでデモを検証する必要がある場合にのみ適用されます。大規模なテスト環境や本番環境での使用は推奨されません。StarRocks に大量のデータをロードするには、[データ取り込みの概要](../loading/Loading_intro.md) を参照して、シナリオに適した他のオプションを検討してください。

ソーステーブルとターゲットテーブルをクエリして、データが存在することを確認します。

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

次の例では、ソーステーブル `source_wiki_edit` を2つの新しい行で上書きします。

```SQL
INSERT OVERWRITE source_wiki_edit
WITH LABEL insert_load_wikipedia_ow
VALUES
    ("2015-09-12 00:00:00","#cn.wikipedia","GELongstreet",0,0,0,0,0,36,36,0),
    ("2015-09-12 00:00:00","#fr.wikipedia","PereBot",0,1,0,1,0,17,17,0);
```

## INSERT OVERWRITE SELECT を使用してデータを上書きする

INSERT OVERWRITE SELECT コマンドを使用して、データソーステーブルに対するクエリの結果でテーブルを上書きできます。INSERT OVERWRITE SELECT 文は、1つ以上の内部または外部テーブルのデータに対してETL操作を実行し、内部テーブルをデータで上書きします。詳細な手順とパラメーターのリファレンスについては、[SQL リファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。

> **注記**
>
> 外部テーブルからのデータのロードは、内部テーブルからのデータのロードと同じです。簡単にするために、以下の例では、内部テーブルのデータでターゲットテーブルを上書きする方法のみを示します。

ソーステーブルとターゲットテーブルをクエリして、異なる行のデータがあることを確認します。

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

- 次の例では、ソーステーブルのデータでテーブル `insert_wiki_edit` を上書きします。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_1
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータでテーブル `insert_wiki_edit` のパーティション `p06` と `p12` を上書きします。

```SQL
INSERT OVERWRITE insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_ow_2
SELECT * FROM source_wiki_edit;
```

ターゲットテーブルをクエリして、データが存在することを確認します。

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

`p06` と `p12` パーティションを切り捨てると、クエリでデータは返されません。

```Plain

```
MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 次の例では、ターゲットテーブル`insert_wiki_edit`をソーステーブルの`event_time`列と`channel`列で上書きします。データが上書きされない列にはデフォルト値が割り当てられます。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

## 生成列を持つテーブルへのデータ挿入

生成列は、定義済みの式または他の列に基づく評価から値が導出される特殊な列です。生成列は、クエリ要求にコストの高い式の評価が含まれる場合（例えば、JSON値から特定のフィールドをクエリする、またはARRAYデータを計算するなど）に特に役立ちます。StarRocksは、データがテーブルにロードされている間に式を評価し、生成された列に結果を保存することで、クエリ時の式の評価を回避し、クエリパフォーマンスを向上させます。

INSERTを使用して、生成列を持つテーブルにデータをロードできます。

次の例では、`insert_generated_columns`テーブルを作成し、そのテーブルに行を挿入します。このテーブルには、`avg_array`と`get_string`の2つの生成列が含まれています。`avg_array`は`data_array`のARRAYデータの平均値を計算し、`get_string`は`data_json`のJSONパス`$.a`から文字列を抽出します。

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

> **注記**
>
> 生成された列へのデータの直接ロードはサポートされていません。

テーブルをクエリして、その中のデータを確認できます。

```Plain
mysql> SELECT * FROM insert_generated_columns;
+------+------------+------------------+-----------+------------+
| id   | data_array | data_json        | avg_array | get_string |
+------+------------+------------------+-----------+------------+
|    1 | [1,2]      | {"a": 1, "b": 2} |       1.5 | 1          |
+------+------------+------------------+-----------+------------+
1 row in set (0.02 sec)
```

## INSERTを使用した非同期データロード

INSERTを使用してデータをロードすると、同期トランザクションが送信されますが、セッションの中断やタイムアウトにより失敗する可能性があります。[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を使用して非同期INSERTトランザクションを送信できます。この機能はStarRocks v2.5以降でサポートされています。

- 次の例では、ソーステーブルからターゲットテーブル`insert_wiki_edit`へデータを非同期に挿入します。

```SQL
SUBMIT TASK AS INSERT INTO insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータでテーブル`insert_wiki_edit`を非同期に上書きします。

```SQL
SUBMIT TASK AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータでテーブル`insert_wiki_edit`を非同期に上書きし、ヒントを使用してクエリタイムアウトを`100000`秒に延長します。

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータでテーブル`insert_wiki_edit`を非同期に上書きし、タスク名を`async`として指定します。

```SQL
SUBMIT TASK async
AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

非同期INSERTタスクのステータスは、Information Schemaのメタデータビュー`task_runs`をクエリすることで確認できます。

次の例では、INSERTタスク`async`のステータスを確認します。

```SQL
SELECT * FROM information_schema.task_runs WHERE task_name = 'async';
```

## INSERTジョブのステータスを確認する

### 結果を通じて確認する

同期INSERTトランザクションは、トランザクションの結果に応じて異なるステータスを返します。

- **トランザクションが成功した場合**

StarRocksは、トランザクションが成功すると以下を返します。

```Plain
Query OK, 2 rows affected (0.05 sec)
{'label':'insert_load_wikipedia', 'status':'VISIBLE', 'txnId':'1006'}
```

- **トランザクションが失敗した場合**

すべてのデータ行がターゲットテーブルにロードできない場合、INSERTトランザクションは失敗します。StarRocksは、トランザクションが失敗した場合、以下を返します。

```Plain
ERROR 1064 (HY000): Insert has filtered data in strict mode, tracking_url=http://x.x.x.x:yyyy/api/_load_error_log?file=error_log_9f0a4fd0b64e11ec_906bbede076e9d08
```

`tracking_url`をチェックすることで問題を特定できます。

### Information Schemaを通じて確認する

[SELECTステートメント](../sql-reference/sql-statements/data-manipulation/SELECT.md)を使用して、`information_schema`データベースの`loads`テーブルから1つ以上のロードジョブの結果をクエリできます。この機能はv3.1以降でサポートされています。

例1: `load_test`データベースで実行されたロードジョブの結果をクエリし、結果を作成時間(`CREATE_TIME`)で降順にソートし、上位の結果のみを返します。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test'
ORDER BY create_time DESC
LIMIT 1\G
```

例2: `load_test`データベースで実行されたロードジョブ（ラベルは`insert_load_wikipedia`）の結果をクエリします。

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test' and label = 'insert_load_wikipedia'\G
```

戻り値は次のとおりです。

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

戻り値のフィールドの詳細については、[Information Schema > loads](../reference/information_schema/loads.md)を参照してください。

### curlコマンドを通じて確認する

INSERTトランザクションのステータスは、curlコマンドを使用して確認できます。

ターミナルを起動し、次のコマンドを実行します。

```Bash
curl --location-trusted -u <username>:<password> \
  http://<fe_address>:<fe_http_port>/api/<db_name>/_load_info?label=<label_name>
```

次の例では、ラベル`insert_load_wikipedia`を使用してトランザクションのステータスを確認します。

```Bash
curl --location-trusted -u <username>:<password> \
  http://x.x.x.x:8030/api/load_test/_load_info?label=insert_load_wikipedia
```

> **注記**
>
> パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力します。

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

INSERTトランザクションには、以下の設定項目を設定できます。

- **FE設定**

| FE設定                             | 説明                                                  |
| ---------------------------------- | ---------------------------------------------------- |
| insert_load_default_timeout_second | INSERTトランザクションのデフォルトタイムアウト。単位は秒です。このパラメータで設定された時間内に現在のINSERTトランザクションが完了しない場合、システムによってキャンセルされ、ステータスはCANCELLEDになります。StarRocksの現在のバージョンでは、このパラメータを使用してすべてのINSERTトランザクションに一律のタイムアウトを指定することしかできず、特定のINSERTトランザクションに異なるタイムアウトを設定することはできません。デフォルトは3600秒（1時間）です。指定した時間内にINSERTトランザクションを完了できない場合は、このパラメータを調整してタイムアウトを延長できます。 |

- **セッション変数**

| セッション変数     | 説明                                                  |
| -------------------- | ---------------------------------------------------- |

| enable_insert_strict | INSERT トランザクションが無効なデータ行を許容するかどうかを制御するスイッチ値です。`true`に設定されている場合、データ行のいずれかが無効であればトランザクションは失敗します。`false`に設定されている場合、少なくとも1行のデータが正しくロードされればトランザクションは成功し、ラベルが返されます。デフォルトは`true`です。この変数は`SET enable_insert_strict = {true or false};`コマンドで設定できます。 |
| query_timeout        | SQL コマンドのタイムアウト時間です。単位は秒です。INSERT という SQL コマンドも、このセッション変数によって制限されます。この変数は`SET query_timeout = xxx;`コマンドで設定できます。 |
