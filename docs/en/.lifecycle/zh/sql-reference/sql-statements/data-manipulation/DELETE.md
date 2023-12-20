---
displayed_sidebar: English
---

|参数|必填|说明|
|---|---|---|
|with_query|否|一个或多个可以在 DELETE 语句中通过名称引用的 CTE。CTE 是可以提高复杂语句可读性的临时结果集。|
|table_name|是|您想要从中删除数据的表。|
|from_item|否|数据库中的一个或多个其他表。这些表可以基于 WHERE 子句中指定的条件与正在操作的表进行连接。基于连接查询的结果集，StarRocks 会从正在操作的表中删除匹配的行。例如，如果 USING 子句是 `USING t1 WHERE t0.pk = t1.pk;`，StarRocks 在执行 DELETE 语句时会将 USING 子句中的表达式转换为 `t0 JOIN t1 ON t0.pk=t1.pk;`。|
|where_condition|是|您想要基于此条件删除行的条件。只有满足 WHERE 条件的行才能被删除。此参数是必需的，因为它有助于防止您意外删除整个表。如果您想删除整个表，可以使用 'WHERE true'.|

### 限制和使用说明

- 主键表不支持从指定分区删除数据，例如，`DELETE FROM <table_name> PARTITION <partition_id> WHERE <where_condition>`。
- 支持以下比较运算符：`=`, `>`, `<`, `>=`, `<=`, `!=`, `IN`, `NOT IN`。
- 支持以下逻辑运算符：`AND` 和 `OR`。
- 您不能使用 DELETE 语句运行并发 DELETE 操作或在数据加载时删除数据。如果执行此类操作，可能无法确保事务的原子性、一致性、隔离性和持久性（ACID）。

### 例子

#### 创建表并插入数据

创建一个名为 `score_board` 的主键表：

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

执行以下语句将数据插入 `score_board` 表：

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

##### 通过主键删除数据

您可以在 DELETE 语句中指定主键，因此 StarRocks 不需要扫描整个表。

从 `score_board` 表中删除 `id` 值为 `0` 的行。

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

##### 通过条件删除数据

示例 1：从 `score_board` 表中删除 `score` 值为 `22` 的行。

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

示例 2：从 `score_board` 表中删除 `score` 值小于 `22` 的行。

```Plain
DELETE FROM score_board WHERE score < 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    3 | Sam  |   22 |
+------+------+------+
```

示例 3：从 `score_board` 表中删除 `score` 值小于 `22` 且 `name` 值不是 `Bob` 的行。

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

##### 通过子查询结果删除数据

您可以在 `DELETE` 语句中嵌套一个或多个子查询，并使用子查询结果作为条件。

在删除数据之前，执行以下语句创建另一个名为 `users` 的表：

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

将数据插入 `users` 表：

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

嵌套一个子查询以找到 `users` 表中 `country` 值为 `China` 的行，并删除 `score_board` 表中与子查询返回的行具有相同 `name` 值的行。您可以使用以下方法之一来实现此目的：

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

##### 使用多表连接或 CTE 删除数据

要删除制片人 "foo" 制作的所有电影，您可以执行以下语句：

```SQL
DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

您还可以使用 CTE 重写上述语句以提高可读性。

```SQL
WITH foo_producers as (
    SELECT * from producers
    where producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```
```
```SQL
DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

您还可以使用CTE重写上述语句以提高可读性。

```SQL
WITH foo_producers AS (
    SELECT * FROM producers
    WHERE producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```
# DELETE

删除表中基于指定条件的数据行。该表可以是分区表或非分区表。

对于 Duplicate Key 表、Aggregate 表和 Unique Key 表，您可以从指定分区删除数据。从 v2.3 版本开始，Primary Key 表支持完整的 `DELETE...WHERE` 语义，允许您基于主键、任何列或子查询的结果删除数据行。从 v3.0 版本开始，StarRocks 通过多表连接和公共表表达式（CTEs）丰富了 `DELETE...WHERE` 语义。如果您需要将 Primary Key 表与数据库中的其他表进行连接，可以在 USING 子句或 CTE 中引用这些其他表。

## 使用说明

- 您必须拥有要执行 DELETE 操作的表和数据库的权限。
- 不建议频繁进行 DELETE 操作。如有需要，请在非高峰时段执行此类操作。
- DELETE 操作只删除表中的数据。表本身仍然存在。要删除表，请运行 [DROP TABLE](../data-definition/DROP_TABLE.md)。
- 为了防止误操作删除整个表中的数据，您必须在 DELETE 语句中指定 WHERE 子句。
- 被删除的行不会立即清理。它们会被标记为“已删除”，并且会暂时保存在 Segment 中。物理上，只有在数据版本合并（压缩）完成后，这些行才会被移除。
- 此操作还会删除引用此表的物化视图中的数据。

## Duplicate Key 表、Aggregate 表和 Unique Key 表

### 语法

```SQL
DELETE FROM  [<db_name>.]<table_name> [PARTITION <partition_name>]
WHERE
column_name1 op { value | value_list } [ AND column_name2 op { value | value_list } ...]
```

### 参数

|**参数**|**是否必须**|**描述**|
|---|---|---|
|`db_name`|否|目标表所属的数据库。如果未指定此参数，默认使用当前数据库。|
|`table_name`|是|您想要从中删除数据的表。|
|`partition_name`|否|您想要从中删除数据的分区。|
|`column_name`|是|您想用作 DELETE 条件的列。您可以指定一个或多个列。|
|`op`|是|DELETE 条件中使用的操作符。支持的操作符有 `=`、`>`、`<`、`>=`、`<=`、`!=`、`IN` 和 `NOT IN`。|

### 限制和使用说明

- 对于 Duplicate Key 表，您可以使用**任何列**作为 DELETE 条件。对于 Aggregate 表和 Unique Key 表，只能使用**关键列**作为 DELETE 条件。

- 您指定的条件必须是 AND 关系。如果您想指定 OR 关系的条件，您必须在单独的 DELETE 语句中指定条件。

- 对于 Duplicate Key 表、Aggregate 表和 Unique Key 表，DELETE 语句不支持使用子查询结果作为条件。

### 影响

在您执行 DELETE 语句后，您的集群的查询性能可能会在一段时间内（在压缩完成之前）下降。性能下降的程度取决于您指定的条件数量。条件数量越多，性能下降的程度越高。

### 示例

#### 创建表并插入数据

以下示例创建了一个分区的 Duplicate Key 表。

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

从 `p1` 分区删除 `k1` 值为 `3` 的行。

```plain
DELETE FROM my_table PARTITION p1
WHERE k1 = 3;

-- 查询结果显示 `k1` 值为 `3` 的行已被删除。

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
+------------+------+------+
```

##### 使用 AND 从指定分区删除数据

从 `p1` 分区删除 `k1` 值大于或等于 `3` 且 `k2` 值为 `"abc"` 的行。

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

从所有分区删除 `k2` 值为 `"abc"` 或 `"cba"` 的行。

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

## Primary Key 表

从 v2.3 版本开始，Primary Key 表支持完整的 `DELETE...WHERE` 语义，允许您基于主键、任何列或子查询删除数据行。

### 语法

```SQL
[ WITH <with_query> [, ...] ]
DELETE FROM <table_name>
[ USING <from_item> [, ...] ]
[ WHERE <where_condition> ]
```

### 参数

|**参数**|**是否必须**|**描述**|
|---|---|---|
|`with_query`|否|一个或多个可以在 DELETE 语句中按名称引用的 CTEs。CTEs 是可以提高复杂语句可读性的临时结果集。|
|`table_name`|是|您想要从中删除数据的表。|
|`from_item`|否|数据库中的一个或多个其他表。这些表可以基于 WHERE 子句中指定的条件与正在操作的表进行连接。基于连接查询的结果集，StarRocks 从正在操作的表中删除匹配的行。例如，如果 USING 子句是 `USING t1 WHERE t0.pk = t1.pk;`，StarRocks 在执行 DELETE 语句时会将 USING 子句中的表达式转换为 `t0 JOIN t1 ON t0.pk=t1.pk;`。|
|`where_condition`|是|您想基于此条件删除行的条件。只有满足 WHERE 条件的行才能被删除。此参数是必需的，因为它有助于防止您意外删除整个表。如果您想删除整个表，可以使用 'WHERE true'。|

### 限制和使用说明

- Primary Key 表不支持从指定分区删除数据，例如 `DELETE FROM <table_name> PARTITION <partition_id> WHERE <where_condition>`。
- 支持以下比较操作符：`=`, `>`, `<`, `>=`, `<=`, `!=`, `IN`, `NOT IN`。
- 支持以下逻辑操作符：`AND` 和 `OR`。
- 您不能使用 DELETE 语句执行并发 DELETE 操作或在数据加载时删除数据。如果执行这些操作，可能无法确保事务的原子性、一致性、隔离性和持久性（ACID）。

### 示例

#### 创建表并插入数据

创建一个名为 `score_board` 的 Primary Key 表：

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

执行以下语句将数据插入 `score_board` 表：

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

##### 通过主键删除数据

您可以在 DELETE 语句中指定主键，因此 StarRocks 不需要扫描整个表。

从 `score_board` 表中删除 `id` 值为 `0` 的行。

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

##### 通过条件删除数据

示例 1：从 `score_board` 表中删除 `score` 值为 `22` 的行。

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

示例 2：从 `score_board` 表中删除 `score` 值小于 `22` 的行。

```Plain
DELETE FROM score_board WHERE score < 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    3 | Sam  |   22 |
+------+------+------+
```

示例 3：从 `score_board` 表中删除 `score` 值小于 `22` 且 `name` 值不是 `Bob` 的行。

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

##### 通过子查询结果删除数据

您可以在 `DELETE` 语句中嵌套一个或多个子查询，并使用子查询结果作为条件。

在您删除数据之前，执行以下语句创建另一个名为 `users` 的表：

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

将数据插入 `users` 表：

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

嵌套子查询以查找 `users` 表中 `country` 值为 `China` 的行，并删除 `score_board` 表中与子查询返回的行具有相同 `name` 值的行。您可以使用以下方法之一来实现这一目的：

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

##### 使用多表连接或 CTE 删除数据

要删除制片人 "foo" 制作的所有电影，您可以执行以下语句：

```SQL
DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

您还可以使用 CTE 重写上述语句以提高可读性。

```SQL
WITH foo_producers as (
    SELECT * from producers
    where producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```