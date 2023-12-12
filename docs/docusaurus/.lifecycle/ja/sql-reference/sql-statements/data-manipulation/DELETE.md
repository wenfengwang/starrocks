---
displayed_sidebar: "Japanese"
---

# DELETE（削除）

指定された条件に基づいてテーブルからデータ行を削除します。テーブルはパーティションされている場合とパーティションされていない場合のどちらも対象となります。

重複キー テーブル、集約 テーブル、ユニーク キー テーブルの場合、特定のパーティションからデータを削除できます。v2.3 以降、プライマリ キー テーブルでは完全な `DELETE...WHERE` セマンティクスがサポートされ、このセマンティクスにより、プライマリ キーや任意の列、あるいはサブクエリの結果に基づいてデータ行を削除できます。v3.0 以降、StarRocks は `DELETE...WHERE` セマンティクスを複数のテーブル結合および共通テーブル式（CTE）で拡張しています。データベース内でプライマリ キー テーブルを他のテーブルと結合する必要がある場合は、USING 句または CTE でこれらの他のテーブルを参照できます。

## 使用上の注意

- 削除を実行するテーブルおよびデータベースに対する権限が必要です。
- 頻繁な DELETE 操作は推奨されません。必要な場合は、オフピーク時に行ってください。
- DELETE 操作はテーブル内のデータのみを削除します。テーブルは削除されません。テーブルを削除するには [DROP TABLE](../data-definition/DROP_TABLE.md) を実行してください。
- テーブル全体のデータを削除するミス操作を防ぐために、DELETE 文で WHERE 句を指定する必要があります。
- 削除された行はすぐにクリーンアップされません。これらは「削除済み」としてマークされ、一時的にセグメントに保存されます。物理的には、データバージョンのマージ（コンパクション）が完了した後にのみ行が削除されます。
- この操作は、このテーブルを参照しているマテリアライズド ビューのデータも削除します。

## 重複キー テーブル、集約 テーブル、ユニーク キー テーブル

### 構文

```SQL
DELETE FROM  [<db_name>.]<table_name> [PARTITION <partition_name>]
WHERE
column_name1 op { value | value_list } [ AND column_name2 op { value | value_list } ...]
```

### パラメータ

| **パラメータ**         | **必須** | **説明**                                                     |
| :--------------- | :------- | :----------------------------------------------------------- |
| `db_name`        | いいえ    | 対象テーブルが属するデータベース。このパラメータを指定しない場合、デフォルトで現在のデータベースが使用されます。      |
| `table_name`     | はい     | データを削除したいテーブル。   |
| `partition_name` | いいえ    | データを削除したいパーティション。  |
| `column_name`    | はい     | DELETE 条件として使用したい列。1 つ以上の列を指定できます。   |
| `op`             | はい     | DELETE 条件で使用される演算子。サポートされている演算子は `=`, `>`, `<`, `>=`, `<=`, `!=`, `IN`, `NOT IN` です。 |

### 制限および使用上の注意

- 重複キー テーブルでは、**任意の列** を削除条件として使用できます。集約テーブルおよびユニーク キー テーブルでは、削除条件として**キー列** のみを使用できます。

- 指定する条件は AND 関係である必要があります。OR 関係で条件を指定する場合は、別々の DELETE 文で条件を指定する必要があります。

- 重複キー テーブル、集約 テーブル、ユニークキー テーブルでは、DELETE 文はサブクエリの結果を条件として使用することはサポートされていません。

### 影響

DELETE 文を実行した後、クラスタのクエリパフォーマンスが一時的に悪化することがあります（コンパクションが完了するまで）。悪化の程度は、指定する条件の数に基づいて異なります。条件の数が多いほど、悪化の程度が高くなります。

### 例

#### テーブルの作成とデータの挿入

以下の例では、パーティションされた重複キー テーブルを作成します。

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

`p1` パーティションから `k1` の値が `3` である行を削除します。

```plain
DELETE FROM my_table PARTITION p1
WHERE k1 = 3;

-- クエリの結果は、`k1` の値が `3` の行が削除されていることを示しています。

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
+------------+------+------+
```

##### AND を使用して特定のパーティションからデータを削除

`p1` パーティションから `k1` の値が `3` 以上で `k2` の値が `"abc"` である行を削除します。

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

すべてのパーティションから `k2` の値が `"abc"` または `"cba"` である行を削除します。

```plain
DELETE FROM my_table
WHERE  k2 in ("abc", "cba");

select * from my_table order by date;
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-12 |    2 | bca  |
+------------+------+------+
```

## プライマリ キー テーブル

v2.3 以降、プライマリ キー テーブルでは完全な `DELETE...WHERE` セマンティクスがサポートされ、このセマンティクスにより、プライマリ キーや任意の列、またはサブクエリに基づいてデータ行を削除できます。

### 構文

```SQL
[ WITH <with_query> [, ...] ]
DELETE FROM <table_name>
[ USING <from_item> [, ...] ]
[ WHERE <where_condition> ]
```

### パラメータ

|   **パラメータ**   | **必須** | **説明**                                              |
| :---------------: | :----------- | :----------------------------------------------------------- |
|   `with_query`    | いいえ     | DELETE 文で名前で参照できる 1 つまたは複数の CTE。CTE は一時的な結果セットであり、複雑なステートメントの可読性を向上させることができます。 |
|   `table_name`    | はい      | データを削除したいテーブル。                |
|    `from_item`    | いいえ     | データベース内の他の 1 つまたは複数のテーブル。これらのテーブルは、WHERE 句で指定された条件に基づいて操作されるテーブルと結合できます。結合クエリの結果セットに基づいて、StarRocks は操作されるテーブルから一致する行を削除します。たとえば、USING 句が `USING t1 WHERE t0.pk = t1.pk;` の場合、StarRocks は DELETE 文を実行するときに、USING 句のテーブル式を `t0 JOIN t1 ON t0.pk=t1.pk;` に変換します。 |
| `where_condition` | はい      | 行を削除したい条件。WHERE 条件を満たす行のみを削除できます。このパラメータは必須ですが、テーブル全体を削除するのを防いでくれます。テーブル全体を削除する場合は 'WHERE true' を使用できます。 |

### 制限および使用上の注意

- プライマリ キー テーブルでは、例えば `DELETE FROM <table_name> PARTITION <partition_id> WHERE <where_condition>` のように特定のパーティションからデータを削除することはサポートされません。
- 次の比較演算子がサポートされています：`=`, `>`, `<`, `>=`, `<=`, `!=`, `IN`, `NOT IN`。
- 次の論理演算子がサポートされています：`AND` および `OR`。
- DELETE文を使用して、同時にDELETE操作を実行したり、データのロード中にデータを削除することはできません。このような操作を行うと、トランザクションの原子性、一貫性、分離性、耐久性（ACID）が確保されない可能性があります。

### 例

#### テーブルの作成とデータの挿入

`score_board`という名前のプライマリキーのテーブルを作成します。

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

次の文を実行して、`score_board`テーブルにデータを挿入します。

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

##### プライマリキーでのデータの削除

`DELETE`文でプライマリキーを指定することができるため、StarRocksはテーブル全体をスキャンする必要がありません。

`score_board`テーブルから`id`の値が`0`の行を削除します。

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

例1：`score_board`テーブルから`score`の値が`22`の行を削除します。

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

例2：`score_board`テーブルから`score`の値が`22`未満の行を削除します。

```Plain
DELETE FROM score_board WHERE score < 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    3 | Sam  |   22 |
+------+------+------+
```

例3：`score_board`テーブルから`score`の値が`22`未満でかつ`name`の値が`Bob`でない行を削除します。

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

##### サブクエリ結果によるデータの削除

`DELETE`文に1つ以上のサブクエリをネストし、サブクエリの結果を条件として使用できます。

データを削除する前に、次の文を実行して`users`という別のテーブルを作成します。

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

`users`テーブルにデータを挿入します。

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

`users`テーブルから`country`の値が`China`である行を見つけるサブクエリをネストし、その結果から返された行と同じ`name`の値を持つ行を`score_board`テーブルから削除します。目的を達成するために、次のいずれかの方法を使用できます。

- 方法1

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

- 方法2

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

##### 複数のテーブルの結合またはCTEを使用したデータの削除

プロデューサーが"foo"で製作されたすべての映画を削除するには、次の文を実行できます。

```SQL
DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

可読性を改善するために、上記の文を再度書き直すためにCTEを使用することもできます。

```SQL
WITH foo_producers as (
    SELECT * from producers
    where producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```