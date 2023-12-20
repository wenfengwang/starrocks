---
displayed_sidebar: English
---

# 删除

根据指定条件从表中删除数据行。表可以是分区表或非分区表。

对于重复键表、聚合表和唯一键表，您可以从指定分区删除数据。从v2.3版本开始，主键表支持完整的DELETE...WHERE语法，允许您基于主键、任何列或子查询结果来删除数据行。从v3.0版本开始，StarRocks通过多表连接和公共表达式（CTE）丰富了DELETE...WHERE语法。如果您需要将主键表与数据库中的其他表进行连接，可以在USING子句或CTE中引用这些其他表。

## 使用须知

- 您必须拥有对您想要执行DELETE操作的表和数据库的权限。
- 不推荐频繁执行DELETE操作。如果需要，请在非高峰时段进行此类操作。
- DELETE操作只会删除表中的数据，而不会删除表本身。要删除整个表，请执行[DROP TABLE](../data-definition/DROP_TABLE.md)命令。
- 为了防止误操作删除整个表中的数据，您必须在DELETE语句中指定WHERE子句。
- 被删除的行并不会立即被清除，而是会被标记为“已删除”，并且会暂时保存在Segment中。物理上，行数据只有在数据版本合并（压缩）完成后才会被移除。
- 此操作还会删除引用此表的物化视图中的数据。

## 重复键表、聚合表和唯一键表

### 语法

```SQL
DELETE FROM  [<db_name>.]<table_name> [PARTITION <partition_name>]
WHERE
column_name1 op { value | value_list } [ AND column_name2 op { value | value_list } ...]
```

### 参数

|参数|必填|说明|
|---|---|---|
|db_name|否|目标表所属的数据库。如果不指定该参数，则默认使用当前数据库。|
|table_name|是|要从中删除数据的表。|
|partition_name|否|要从中删除数据的分区。|
|column_name|Yes|要用作 DELETE 条件的列。您可以指定一列或多列。|
|op|Yes|DELETE 条件中使用的运算符。支持的运算符包括 =、>、<、>=、<=、!=、IN 和 NOT IN。|

### 限制和使用须知

- 对于重复键表，您可以使用**任何列**作为删除条件。对于聚合表和唯一键表，只有**键列**可以用作删除条件。

- 您指定的条件必须是AND关系。如果您想要指定OR关系中的条件，必须在单独的DELETE语句中指定这些条件。

- 对于重复键表、聚合表和唯一键表，DELETE语句不支持使用子查询结果作为条件。

### 影响

执行DELETE语句后，您的集群查询性能可能会在一段时间内（压缩完成前）下降。性能下降的程度取决于您指定的条件数量。条件数量越多，性能下降的程度越大。

### 示例

#### 创建表并插入数据

以下示例创建了一个分区的重复键表。

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

从p1分区删除k1值为3的行。

```plain
DELETE FROM my_table PARTITION p1
WHERE k1 = 3;

-- The query result shows rows whose `k1` values are `3` are deleted.

select * from my_table partition (p1);
+------------+------+------+
| date       | k1   | k2   |
+------------+------+------+
| 2022-03-11 |    2 | acb  |
| 2022-03-11 |    4 | abc  |
+------------+------+------+
```

##### 使用AND从指定分区删除数据

从p1分区删除k1值大于或等于3并且k2值为“abc”的行。

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

从所有分区删除k2值为“abc”或“cba”的行。

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

从v2.3版本开始，主键表支持完整的DELETE...WHERE语法，允许您根据主键、任意列或子查询删除数据行。

### 语法

```SQL
[ WITH <with_query> [, ...] ]
DELETE FROM <table_name>
[ USING <from_item> [, ...] ]
[ WHERE <where_condition> ]
```

### 参数

|参数|必填|说明|
|---|---|---|
|with_query|否|可以在 DELETE 语句中按名称引用的一个或多个 CTE。 CTE 是临时结果集，可以提高复杂语句的可读性。|
|table_name|是|要从中删除数据的表。|
|from_item|否|数据库中的一个或多个其他表。这些表可以根据 WHERE 子句中指定的条件与正在操作的表进行连接。 StarRocks 根据连接查询的结果集，从正在操作的表中删除匹配的行。例如，如果 USING 子句为 USING t1 WHERE t0.pk = t1.pk;，StarRocks 会将 USING 子句中的表表达式转换为 t0 JOIN t1 ON t0.pk=t1.pk;执行 DELETE 语句时。|
|where_condition|Yes|要删除行所依据的条件。只有满足WHERE条件的行才能被删除。此参数是必需的，因为它有助于防止您意外删除整个表。如果要删除整个表，可以使用'WHERE true'。|

### 限制和使用须知

- 主键表不支持从指定分区删除数据，例如DELETE FROM <table_name> PARTITION <partition_id> WHERE <where_condition>。
- 支持以下比较操作符：=、>、<、>=、<=、!=、IN、NOT IN。
- 支持以下逻辑操作符：AND和OR。
- 您不能使用DELETE语句进行并发的DELETE操作或在数据加载时删除数据。如果进行此类操作，可能无法确保事务的原子性、一致性、隔离性和持久性（ACID）。

### 示例

#### 创建表并插入数据

创建一个名为score_board的主键表：

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

执行以下语句向score_board表插入数据：

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

您可以在DELETE语句中指定主键，这样StarRocks就不需要扫描整个表。

从score_board表中删除id值为0的行。

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

示例1：从score_board表中删除score值为22的行。

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

示例2：从score_board表中删除score值小于22的行。

```Plain
DELETE FROM score_board WHERE score < 22;

select * from score_board;
+------+------+------+
| id   | name | score|
+------+------+------+
|    3 | Sam  |   22 |
+------+------+------+
```

示例3：从score_board表中删除score值小于22且name值不是Bob的行。

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

您可以在DELETE语句中嵌入一个或多个子查询，并使用子查询结果作为条件。

在删除数据之前，执行以下语句创建另一个名为users的表：

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

向users表中插入数据：

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

嵌入子查询以找到users表中country值为China的行，并从score_board表中删除与子查询返回的行具有相同name值的行。您可以使用以下任一方法实现该目的：

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

##### 使用多表连接或CTE删除数据

要删除制片人“foo”制作的所有电影，您可以执行以下语句：

```SQL
DELETE FROM films USING producers
WHERE producer_id = producers.id
    AND producers.name = 'foo';
```

您也可以使用CTE重写上述语句，以提高语句的可读性。

```SQL
WITH foo_producers as (
    SELECT * from producers
    where producers.name = 'foo'
)
DELETE FROM films USING foo_producers
WHERE producer_id = foo_producers.id;
```
