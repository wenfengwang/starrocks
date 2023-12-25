---
displayed_sidebar: Chinese
---

# DELETE

この文は、テーブルからデータ行を削除するために使用されます。テーブルは、パーティションテーブルまたは非パーティションテーブルのいずれかです。

明細型（Duplicate Key）、集約型（Aggregate Key）、および更新型テーブル（Unique Key）では、**指定されたパーティション**のデータを削除することができます。バージョン2.3から、主キー型テーブルは完全なDELETE...WHERE構文をサポートし、主キー、任意の列、およびサブクエリの結果に基づいてデータを削除できます。バージョン3.0からは、主キー型テーブルはDELETE...WHERE構文を拡張し、複数テーブルの結合や共通テーブル式（CTE）を使用することができます。

## 注意事項

- DELETE操作を実行するには、対象テーブルのDELETE権限が必要です。
- 高頻度のDELETE操作は推奨されません。必要な場合は、業務のオフピーク時に実施してください。
- DELETEはテーブルのデータを削除しますが、テーブル自体は残ります。テーブルを削除する場合は、[DROP TABLE](../data-definition/DROP_TABLE.md)を参照してください。
- 誤ってテーブル全体を削除することを防ぐために、DELETE文にはWHERE句を指定する必要があります。
- 削除されたデータは「Deleted」としてマークされ、一時的にセグメント内に保持され、物理削除はすぐには行われません。コンパクション（データバージョンのマージ）が完了した後に回収されます。
- この操作は、テーブルに関連するマテリアライズドビューのデータも同時に削除します。

## DELETEと明細型、集約型、更新型テーブル

### 構文

```SQL
DELETE FROM [<db_name>.]<table_name> [PARTITION <partition_name>]
WHERE
column_name1 op { value | value_list } [ AND column_name2 op { value | value_list } ...]
```

### パラメータ説明

| **パラメータ**   | **必須** | **説明**                                                     |
| ---------------- | -------- | ------------------------------------------------------------ |
| `db_name`        | いいえ       | 操作対象のテーブルがあるデータベース。指定しない場合は、現在のデータベースがデフォルトです。|
| `table_name`     | はい      | 操作対象のテーブル名。                                       |
| `partition_name` | いいえ       | 操作対象のパーティション名。                                 |
| `column_name`    | はい      | 削除条件として使用される列。1つまたは複数の列を指定できます。|
| `op`             | はい      | 使用される演算子。`=`、`>`、`<`、`>=`、`<=`、`!=`、`IN`、`NOT IN`をサポートします。|

### 使用制限

- 集約型および更新型テーブルは**キー列のみ**を削除条件としてサポートしています。明細型テーブルは**任意の列**を削除条件としてサポートしています。
- 条件は「AND」の関係のみです。「OR」の関係を実現したい場合は、条件を別々のDELETE文に分けて記述する必要があります。
- 明細型、集約型、更新型テーブルでは、現在DELETE文でサブクエリの結果を削除条件として使用することはサポートされていません。

### 影響

DELETE文を実行すると、コンパクションが完了するまでの間（一時的に）クエリの効率が低下する可能性があります。影響の程度は、文中で指定された削除条件の数に依存します。条件が多いほど、影響は大きくなります。

### 例

#### テーブルの作成とデータの挿入

以下の例では、明細型のパーティションテーブルを作成します。

```SQL
CREATE TABLE `my_table` (
    `date` date NOT NULL,
    `k1` int(11) NOT NULL COMMENT "",
    `k2` varchar(65533) NULL DEFAULT "" COMMENT "")
DUPLICATE KEY(`date`)
PARTITION BY RANGE(`date`)
(
    PARTITION p1 VALUES [('2022-03-11'), ('2022-03-12')),
    PARTITION p2 VALUES [('2022-03-12'), ('2022-03-13'))
)
DISTRIBUTED BY HASH(`date`)
PROPERTIES
("replication_num" = "3");

INSERT INTO `my_table` VALUES
('2022-03-11', 3, 'abc'),
('2022-03-11', 2, 'acb'),
('2022-03-11', 4, 'abc'),
('2022-03-12', 2, 'bca'),
('2022-03-12', 4, 'cba'),
('2022-03-12', 5, 'cba');
```

#### テーブルデータのクエリ

```SQL
select * from my_table order by date; 
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    3 | abc  |
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
| 2022-03-12 |    2 | bca  |
| 2022-03-12 |    4 | cba  |
| 2022-03-12 |    5 | cba  |
+------------+------+------+
```

#### データの削除

**指定されたパーティション内の行を削除**

`my_table`テーブルの`p1`パーティション内で`k1`列の値が`3`のデータ行を削除します。

```SQL
DELETE FROM my_table PARTITION p1
WHERE k1 = 3;

-- p1パーティション内でk1が3の行が削除されたことがわかります。
select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
+------------+------+------+
```

**AND条件を満たす指定されたパーティション内の行を削除**

`my_table`テーブルの`p1`パーティション内で`k1`列の値が`3`以上で、かつ`k2`列の値が`"abc"`のデータ行を削除します。

```SQL
DELETE FROM my_table PARTITION p1
WHERE k1 >= 3 AND k2 = "abc";

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
+------------+------+------+
```

**条件を満たすすべてのパーティション内の行を削除**

`my_table`テーブルのすべてのパーティション内で`k2`列の値が`"abc"`または`"cba"`のデータ行を削除します。

```SQL
DELETE FROM my_table
WHERE k2 in ("abc", "cba");

select * from my_table order by date;
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-12 |    2 | bca  |
+------------+------+------+
```

## 主キー型テーブルとのDELETE

バージョン2.3から、主キー型テーブルは完全なDELETE...WHERE構文をサポートしています。主キー、任意の列、およびサブクエリの結果に基づいてデータを削除することができます。バージョン3.0からは、StarRocksはDELETE...WHERE構文を拡張し、複数テーブルの結合や共通テーブル式（CTE）を使用することができます。操作対象のテーブルをデータベース内の他のテーブルと結合する必要がある場合は、USING句またはCTEで他のテーブルを参照できます。

### 構文

```SQL
[ WITH <with_query> [, ...] ]
DELETE FROM <table_name>

[ USING <from_item> [, ...] ]
[ WHERE <where_condition> ]
```

### パラメータ説明

| パラメータ        | 必須 | 説明                                                         |
| --------------- | ---- | ------------------------------------------------------------ |
| with_query      | いいえ  | DELETE文で名前を指定して参照できる1つ以上のCTE。CTEは一時的な結果セットで、複雑な文の可読性を向上させることができます。 |
| table_name      | はい  | 操作対象のテーブル名。                                       |
| from_item       | いいえ  | データベース内の1つまたは複数の他のテーブルを参照します。このテーブルは操作対象のテーブルと結合され、WHERE句で結合条件が指定され、最終的にStarRocksは結合クエリの結果セットに基づいて操作対象のテーブルから一致する行を削除します。例えばUSING句が `USING t1 WHERE t0.pk = t1.pk;` と指定された場合、StarRocksは実際にDELETE文を実行する際に、このUSING句のテーブル式を `t0 JOIN t1 ON t0.pk=t1.pk;` に変換します。 |
| where_condition | いいえ  | WHERE条件を満たす行のみが削除されます。このパラメータは必須で、誤ってテーブル全体を削除することを防ぎます。テーブル内のすべての行を削除するには、`WHERE true` を使用してください。 |

### 注意事項

- 主キータイプのテーブルは現在、特定のパーティション内のデータを削除することは**サポートされていません**。例：`DELETE FROM <table_name> PARTITION <partition_id> WHERE <where_condition>`。
- 次の比較演算子をサポートしています：`=`、`>`、`<`、`>=`、`<=`、`!=`、`IN`、`NOT IN`。
- 次の論理演算子をサポートしています：`AND`、`OR`。
- 削除とデータの同時インポートはサポートされていません。これは、インポートのトランザクション性を保証できない可能性があるためです。

### 例

#### テーブルの作成とデータの挿入

`test` データベースに `score_board` という名前の主キータイプのテーブルを作成します：

```SQL
CREATE TABLE `score_board` (
    `id` int(11) NOT NULL COMMENT "",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT "")
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`)
PROPERTIES
(
    "replication_num" = "3",
    "storage_format" = "DEFAULT",
    "enable_persistent_index" = "false"
);

INSERT INTO score_board VALUES
(0, 'Jack', 21),
(1, 'Bob', 21),
(2, 'Stan', 21),
(3, 'Sam', 22);
```

#### テーブルデータのクエリ

```Plain
select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    0 | Jack |   21 |
|    1 | Bob  |   21 |
|    2 | Stan |   21 |
|    3 | Sam  |   22 |
+------+------+------+
4 rows in set (0.00 sec)
```

#### データの削除

**主キーによるデータ削除**

主キーを指定することで、全表スキャンを避けることができます。

`score_board` テーブルの `id` 列の値が `0` のデータ行を削除します。

```SQL
DELETE FROM score_board WHERE id = 0;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    1 | Bob  |   21 |
|    2 | Stan |   21 |
|    3 | Sam  |   22 |
+------+------+------+
3 rows in set (0.01 sec)
```

**条件によるデータ削除**

条件には任意の列を使用できます。

例1：`score_board` テーブルの `score` 列の値が `22` に等しいすべてのデータ行を削除します。

```SQL
DELETE FROM score_board WHERE score = 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    0 | Jack |   21 |
|    1 | Bob  |   21 |
|    2 | Stan |   21 |
+------+------+------+
3 rows in set (0.01 sec)
```

例2：`score_board` テーブルの `score` 列の値が `22` 未満のすべてのデータ行を削除します。

```SQL
DELETE FROM score_board WHERE score < 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    3 | Sam  |   22 |
+------+------+------+
1 row in set (0.01 sec)
```

例3：`score_board` テーブルの `score` 列の値が `22` 未満で、かつ `name` 列の値が `Bob` ではないすべてのデータ行を削除します。

```SQL
DELETE FROM score_board WHERE score < 22 and name != "Bob";

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    1 | Bob  |   21 |
|    3 | Sam  |   22 |
+------+------+------+
2 rows in set (0.00 sec)
```

**サブクエリの結果に基づくデータ削除**

`DELETE` 文に1つ以上のサブクエリをネストし、サブクエリの結果を削除条件として使用できます。

削除操作を開始する前に、`test` データベースに `users` という名前の主キータイプのテーブルを作成します：

```SQL
CREATE TABLE
    `users` (`uid` int(11) NOT NULL COMMENT "",
    `name` varchar(65533) NOT NULL COMMENT "",
    `country` varchar(65533) NULL COMMENT "")
ENGINE=OLAP
PRIMARY KEY(`uid`)COMMENT "OLAP"
DISTRIBUTED BY HASH(`uid`)
PROPERTIES
(
    "replication_num" = "3",
    "storage_format" = "DEFAULT",
    "enable_persistent_index" = "false"
);
```

`users` テーブルにデータを挿入します：

```SQL
INSERT INTO users VALUES
(0, "Jack", "China"),
(2, "Stan", "USA"),
(1, "Bob", "China"),
(3, "Sam", "USA");
select * from users;
+------+------+---------+
| uid  | name | country |
+------+------+---------+
|    0 | Jack | China   |
|    1 | Bob  | China   |
|    2 | Stan | USA     |
|    3 | Sam  | USA     |
+------+------+---------+
4 rows in set (0.00 sec)
```

`users` テーブルで `country` 列の値が `China` のデータ行を検索し、その後 `score_board` テーブルで `users` テーブルで検索されたデータ行と `name` 列の値が同じすべてのデータ行を削除します。実装方法は2つあります：

- 方法 1

```SQL
DELETE FROM score_board
WHERE name IN (select name from users where country = "China");
select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    2 | Stan |   21 |
|    3 | Sam  |   22 |
+------+------+------+
2 rows in set (0.00 sec)
```

- 方法 2

```SQL
DELETE FROM score_board
WHERE EXISTS (select name from users
              where score_board.name = users.name and country = "China");
select * from score_board;
+------+------+-------+
| id   | name | score |
+------+------+-------+
|    2 | Stan |    21 |
|    3 | Sam  |    22 |
+------+------+-------+
2 rows in set (0.00 sec)
```

**複数のテーブルを結合したり、CTEの結果セットに基づいてデータを削除する**

プロデューサーfooが制作したすべての映画を削除するには、以下のステートメントを実行します：

```SQL

DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

上記のステートメントは、CTEを使用して書き換えることで、より読みやすくすることができます。

```SQL
WITH foo_producers AS (
    SELECT * FROM producers
    WHERE producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```
