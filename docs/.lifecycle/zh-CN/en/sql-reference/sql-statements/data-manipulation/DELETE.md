---
displayed_sidebar: "Chinese"
---

# DELETE

根据指定的条件，从表中删除数据行。表可以是分区表或非分区表。

对于**重复键表**、**聚合表**和**唯一键表**，您可以从指定的分区删除数据。从 v2.3 开始，**主键表** 支持完整的`DELETE...WHERE`语义，允许您根据主键、任何列或子查询的结果删除数据行。从 v3.0 开始，StarRocks 在`DELETE...WHERE`语义中丰富了多表连接和公共表表达式（CTE）。如果需要将主键表与数据库中的其他表连接，可以在 USING 子句或 CTE 中引用这些其他表。

## 使用说明

- 您必须拥有对要执行 DELETE 操作的表和数据库的权限。
- 不建议频繁执行 DELETE 操作。如果需要，请在非高峰时间执行此类操作。
- DELETE 操作仅删除表中的数据。表保留不变。要删除表，请运行[Dropping Tables](../data-definition/DROP_TABLE.md)。
- 为防止误操作删除整个表中的数据，必须在 DELETE 语句中指定 WHERE 子句。
- 已删除的行不会立即清除。它们被标记为“已删除”，将临时保存在节段中。在数据版本合并（压缩）完成后，这些行才会被物理删除。
- 此操作还会删除引用此表的物化视图的数据。

## 重复键表，聚合表和唯一键表

### 语法

```SQL
DELETE FROM  [<db_name>.]<table_name> [PARTITION <partition_name>]
WHERE
column_name1 op { value | value_list } [ AND column_name2 op { value | value_list } ...]
```

### 参数

| **参数**         | **必需** | **描述**                                                     |
| :--------------- | :------- | :----------------------------------------------------------- |
| `db_name`        | 否       | 目标表所属的数据库。如果未指定此参数，默认使用当前数据库。      |
| `table_name`     | 是       | 您要从中删除数据的表。   |
| `partition_name` | 否       | 要从中删除数据的分区。  |
| `column_name`    | 是       | 您要用作删除条件的列。可以指定一个或多个列。   |
| `op`             | 是       | 删除条件中要使用的运算符。支持的运算符有`=`、`>`、`<`、`>=`、`<=`、`!=`、`IN` 和 `NOT IN`。 |

### 限制和使用说明

- 对于**重复键表**，您可以使用**任何列**作为删除条件。对于**聚合表**和**唯一键表**，只能使用**键列**作为删除条件。

- 您指定的条件必须是 AND 关系。如果想指定 OR 关系的条件，必须在单独的 DELETE 语句中指定条件。

- 对于**重复键表**、**聚合表**和**唯一键表**，DELETE 语句不支持将子查询结果用作条件。

### 影响

执行 DELETE 语句后，一段时间内（在压缩完成之前），集群的查询性能可能会下降。下降程度取决于您指定的条件数量。条件数量越多，性能下降越严重。

### 示例

#### 创建表并插入数据

以下示例创建了一个分区**重复键表**。

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

#### 查询数据

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

#### 删除数据

##### 从指定分区删除数据

删除`p1`分区中`k1`值为`3`的行。

```plain
DELETE FROM my_table PARTITION p1
WHERE k1 = 3;

-- 查询结果显示，`k1`值为`3`的行已删除。

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
+------------+------+------+
```

##### 使用 AND 从指定分区删除数据

删除`p1`分区中`k1`值大于或等于`3`且`k2`值为`"abc"`的行。

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

##### 从所有分区删除数据

删除所有分区中`k2`值为`"abc"`或`"cba"`的行。

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

## 主键表

从 v2.3 开始，主键表支持完整的`DELETE...WHERE`语义，允许您根据主键、任何列或子查询的结果删除数据行。

### 语法

```SQL
[ WITH <with_query> [, ...] ]
DELETE FROM <table_name>
[ USING <from_item> [, ...] ]
[ WHERE <where_condition> ]
```

### 参数

|   **参数**   | **必需** | **描述**                                              |
| :----------- | :------- | :----------------------------------------------------- |
|   `with_query`| 否       | 可在 DELETE 语句中按名称引用的一个或多个 CTE。CTE 是临时结果集，可以提高复杂语句的可读性。 |
|   `table_name`| 是       | 您要从中删除数据的表。                                  |
|    `from_item`| 否       | 数据库中的一个或多个其他表。这些表可以根据 WHERE 子句中指定的条件与被操作表进行连接。根据联接查询的结果集，StarRocks 从被操作表中删除匹配的行。例如，如果 USING 子句为`USING t1 WHERE t0.pk = t1.pk;`，StarRocks 在执行 DELETE 语句时会将 USING 子句中的表达式转换为` t0 JOIN t1 ON t0.pk=t1.pk;`。 |
| `where_condition` | 是       | 您希望根据其来删除行的条件。只能删除满足 WHERE 条件的行。此参数是必需的，因为它有助于防止您意外删除整个表。如果要删除整个表，可以使用'WHERE true'。|

### 限制和使用说明

- 主键表不支持从指定分区删除数据，例如`DELETE FROM <table_name> PARTITION <partition_id> WHERE <where_condition>`。
- 支持以下比较运算符：`=`、`>`、`<`、`>=`、`<=`、`!=`、`IN`、`NOT IN`。
- 支持以下逻辑运算符：`AND` 和 `OR`。
- 您不能使用DELETE语句运行并发DELETE操作或在数据加载时删除数据。如果执行此类操作，则可能无法确保事务的原子性、一致性、隔离性和持久性（ACID）。

### 示例

#### 创建表并插入数据

创建名为`score_board`的主键表：

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

#### 查询数据

执行以下语句将数据插入`score_board`表：

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

#### 删除数据

##### 按主键删除数据

您可以在DELETE语句中指定主键，因此StarRocks无需扫描整个表。

删除`score_board`表中`id`值为`0`的行。

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

##### 按条件删除数据

示例1：从`score_board`表中删除`score`值为`22`的行。

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

示例2：从`score_board`表中删除`score`值小于`22`的行。

```Plain
DELETE FROM score_board WHERE score < 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    3 | Sam  |   22 |
+------+------+------+
```

示例3：从`score_board`表中删除`score`值小于`22`且`name`值不是`Bob`的行。

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

##### 按子查询结果删除数据

您可以在`DELETE`语句中嵌套一个或多个子查询，并将子查询结果用作条件。

在删除数据之前，执行以下语句创建另一个名为`users`的表：

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

向`users`表中插入数据：

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

嵌套一个子查询，从`users`表中查找`country`值为`China`的行，并删除具有与子查询返回的行相同`name`值的行，从`score_board`表中。您可以使用以下其中一种方法来实现目的：

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

##### 通过使用多表连接或CTE删除数据

要删除所有由制片人“foo”制作的电影，可以执行以下语句：

```SQL
DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

您还可以使用CTE改写上述语句以提高可读性。

```SQL
WITH foo_producers as (
    SELECT * from producers
    where producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```