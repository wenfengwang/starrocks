---
displayed_sidebar: English
---

# DELETE

指定された条件に基づいてテーブルからデータ行を削除します。テーブルはパーティション分割されたものでも非パーティション分割されたものでもかまいません。

Duplicate Key テーブル、Aggregate テーブル、および Unique Key テーブルでは、指定されたパーティションからデータを削除することができます。v2.3 から、Primary Key テーブルは完全な `DELETE...WHERE` セマンティクスをサポートし、主キー、任意のカラム、またはサブクエリの結果に基づいてデータ行を削除することができます。v3.0 から、StarRocks は `DELETE...WHERE` セマンティクスをマルチテーブル結合と共通テーブル式 (CTE) で拡張しています。データベース内の他のテーブルと Primary Key テーブルを結合する必要がある場合、USING 句または CTE でこれらの他のテーブルを参照することができます。

## 使用上の注意

- DELETE を実行したいテーブルとデータベースに対する権限が必要です。
- 頻繁な DELETE 操作は推奨されません。必要な場合は、オフピーク時間に操作を行ってください。
- DELETE 操作はテーブル内のデータのみを削除します。テーブル自体は残ります。テーブルを削除するには、[DROP TABLE](../data-definition/DROP_TABLE.md) を実行してください。
- 誤操作によるテーブル全体のデータ削除を防ぐため、DELETE ステートメントには WHERE 句を指定する必要があります。
- 削除された行は直ちにはクリーンアップされません。それらは「削除済み」としてマークされ、セグメント内に一時的に保存されます。物理的には、データバージョンのマージ（コンパクション）が完了した後にのみ行が削除されます。
- この操作は、このテーブルを参照するマテリアライズドビューのデータも削除します。

## Duplicate Key テーブル、Aggregate テーブル、および Unique Key テーブル

### 構文

```SQL
DELETE FROM [<db_name>.]<table_name> [PARTITION <partition_name>]
WHERE
column_name1 op { value | value_list } [ AND column_name2 op { value | value_list } ...]
```

### パラメータ

| **パラメータ**         | **必須** | **説明**                                                     |
| :--------------- | :------- | :----------------------------------------------------------- |
| `db_name`        | いいえ       | 対象テーブルが属するデータベース。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。      |
| `table_name`     | はい      | データを削除するテーブル。   |
| `partition_name` | いいえ       | データを削除するパーティション。  |
| `column_name`    | はい      | DELETE 条件として使用するカラム。一つ以上のカラムを指定できます。   |
| `op`             | はい      | DELETE 条件で使用される演算子。サポートされている演算子は `=`, `>`, `<`, `>=`, `<=`, `!=`, `IN`, `NOT IN` です。 |

### 制限と使用上の注意

- Duplicate Key テーブルでは、**任意のカラム**を DELETE 条件として使用できます。Aggregate テーブルと Unique Key テーブルでは、DELETE 条件として**キーカラムのみ**使用できます。

- 指定する条件は AND 関係でなければなりません。OR 関係で条件を指定したい場合は、別々の DELETE ステートメントで条件を指定する必要があります。

- Duplicate Key テーブル、Aggregate テーブル、および Unique Key テーブルでは、DELETE ステートメントでサブクエリの結果を条件として使用することはサポートされていません。

### 影響

DELETE ステートメントを実行すると、コンパクションが完了する前の一定期間、クラスタのクエリパフォーマンスが低下する可能性があります。パフォーマンスの低下の程度は、指定した条件の数によって異なります。条件の数が多いほど、パフォーマンスの低下の度合いが高くなります。

### 例

#### テーブルの作成とデータの挿入

以下の例では、パーティション分割された Duplicate Key テーブルを作成します。

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

#### データのクエリ

```plain
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

##### 特定のパーティションからデータを削除

`p1` パーティションから `k1` の値が `3` の行を削除します。

```plain
DELETE FROM my_table PARTITION p1
WHERE k1 = 3;

-- クエリ結果は `k1` の値が `3` の行が削除されたことを示します。

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
+------------+------+------+
```

##### AND を使用して特定のパーティションからデータを削除

`p1` パーティションから `k1` の値が `3` 以上で `k2` の値が `"abc"` の行を削除します。

```plain
DELETE FROM my_table PARTITION p1
WHERE k1 >= 3 AND k2 = "abc";

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
+------------+------+------+
```

##### すべてのパーティションからデータを削除

`k2` の値が `"abc"` または `"cba"` の行をすべてのパーティションから削除します。

```plain
DELETE FROM my_table
WHERE k2 IN ("abc", "cba");

select * from my_table order by date;
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-12 |    2 | bca  |
+------------+------+------+
```

## Primary Key テーブル

v2.3 から、Primary Key テーブルは完全な `DELETE...WHERE` セマンティクスをサポートし、主キー、任意のカラム、またはサブクエリに基づいてデータ行を削除することができます。


### 構文

```SQL
[ WITH <with_query> [, ...] ]
DELETE FROM <table_name>
[ USING <from_item> [, ...] ]
[ WHERE <where_condition> ]
```

### パラメーター

|   **パラメーター**   | **必須** | **説明**                                              |
| :---------------: | :----------- | :----------------------------------------------------------- |
|   `with_query`    | いいえ           | DELETE ステートメントで名前で参照できる 1 つ以上の CTE。CTE は、複雑なステートメントの可読性を向上させる一時的な結果セットです。 |
|   `table_name`    | はい          | データを削除したいテーブル。                |
|    `from_item`    | いいえ           | データベース内の他のテーブル 1 つ以上。これらのテーブルは、WHERE 句で指定された条件に基づいて操作対象のテーブルと結合できます。結合クエリの結果セットに基づいて、StarRocks は操作対象のテーブルから一致した行を削除します。例えば、USING 句が `USING t1 WHERE t0.pk = t1.pk;` の場合、StarRocks は DELETE ステートメントを実行する際に USING 句のテーブル式を `t0 JOIN t1 ON t0.pk=t1.pk;` に変換します。 |
| `where_condition` | はい          | 行を削除する条件。WHERE 条件を満たす行のみが削除されます。このパラメーターは必須で、誤ってテーブル全体を削除するのを防ぐために役立ちます。テーブル全体を削除したい場合は、'WHERE true' を使用できます。 |

### 制限と使用上の注意

- 主キー テーブルは、特定のパーティションからデータを削除することをサポートしていません。例えば `DELETE FROM <table_name> PARTITION <partition_id> WHERE <where_condition>` は使用できません。
- サポートされている比較演算子には `=`, `>`, `<`, `>=`, `<=`, `!=`, `IN`, `NOT IN` があります。
- サポートされている論理演算子には `AND` と `OR` があります。
- DELETE ステートメントを使用して同時に DELETE 操作を行ったり、データロード時にデータを削除したりすることはできません。そのような操作を行うと、トランザクションの ACID（原子性、一貫性、分離性、耐久性）が保証されない可能性があります。

### 例

#### テーブルの作成とデータの挿入

`score_board` という名前の主キー テーブルを作成します。

```sql
CREATE TABLE `score_board` (
  `id` int(11) NOT NULL COMMENT "",
  `name` varchar(65533) NULL DEFAULT "" COMMENT "",
  `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
) ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
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

#### データのクエリ

`score_board` テーブルにデータを挿入するために、以下のステートメントを実行します。

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

##### 主キーによるデータの削除

DELETE ステートメントで主キーを指定することにより、StarRocks はテーブル全体をスキャンする必要がありません。

`score_board` テーブルから `id` の値が `0` の行を削除します。

```Plain
DELETE FROM score_board WHERE id = 0;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    1 | Bob  |   21 |
|    2 | Stan |   21 |
|    3 | Sam  |   22 |
+------+------+------+
```

##### 条件によるデータの削除

例 1: `score_board` テーブルから `score` の値が `22` の行を削除します。

```Plain
DELETE FROM score_board WHERE score = 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    0 | Jack |   21 |
|    1 | Bob  |   21 |
|    2 | Stan |   21 |
+------+------+------+
```

例 2: `score_board` テーブルから `score` の値が `22` 未満の行を削除します。

```Plain
DELETE FROM score_board WHERE score < 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    3 | Sam  |   22 |
+------+------+------+
```

例 3: `score_board` テーブルから `score` の値が `22` 未満で、かつ `name` の値が `Bob` でない行を削除します。

```Plain
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

##### サブクエリの結果によるデータの削除

`DELETE` ステートメントに 1 つ以上のサブクエリをネストし、サブクエリの結果を条件として使用することができます。

データを削除する前に、`users` という名前の別のテーブルを作成するために、以下のステートメントを実行します。

```SQL
CREATE TABLE `users` (
  `uid` int(11) NOT NULL COMMENT "",
  `name` varchar(65533) NOT NULL COMMENT "",
  `country` varchar(65533) NULL COMMENT ""
) ENGINE=OLAP
PRIMARY KEY(`uid`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`uid`)
PROPERTIES (
"replication_num" = "3",
"storage_format" = "DEFAULT",
"enable_persistent_index" = "false"
);
```

`users` テーブルにデータを挿入します。

```SQL
INSERT INTO users VALUES
(0, "Jack", "China"),
(2, "Stan", "USA"),
(1, "Bob", "China"),
(3, "Sam", "USA");
```

```plain
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

`users` テーブルから `country` の値が `China` の行を見つけるサブクエリをネストし、サブクエリから返された行と同じ `name` の値を持つ行を `score_board` テーブルから削除します。目的を達成するために、以下の方法のいずれかを使用できます。

- 方法 1

```plain
DELETE FROM score_board
WHERE name IN (select name from users where country = "China");
    
select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    2 | Stan |   21 |
|    3 | Sam  |   22 |
+------+------+------+
```

- 方法 2

```plain
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

##### マルチテーブル結合または CTE を使用してデータを削除する

プロデューサー "foo" によって制作されたすべての映画を削除するには、以下のステートメントを実行します。

```SQL
DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

CTEを使用して上記のステートメントを書き換え、可読性を向上させることができます。

```SQL
WITH foo_producers AS (
    SELECT * FROM producers
    WHERE producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```

