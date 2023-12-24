---
displayed_sidebar: English
---

# STRUCT

## 描述

STRUCT 用于表示复杂的数据类型。它表示具有不同数据类型的元素（也称为字段）的集合，例如 `<a INT, b STRING>`。

结构中的字段名称必须是唯一的。字段可以是原始数据类型（如数字、字符串或日期）或复杂数据类型（如 ARRAY 或 MAP）。

结构中的字段也可以是另一个 STRUCT、ARRAY 或 MAP，这允许您创建嵌套数据结构，例如 `STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>`。

从 v3.1 开始支持 STRUCT 数据类型。在 v3.1 版本中，您可以在创建 StarRocks 表时定义 STRUCT 列，将 STRUCT 数据加载到该表中，并查询 MAP 数据。

从 v2.5 开始，StarRocks 支持从数据湖中查询复杂数据类型 MAP 和 STRUCT。您可以使用 StarRocks 提供的外部目录查询 Apache Hive™、Apache Hudi 和 Apache Iceberg 的 MAP 和 STRUCT 数据。您只能查询 ORC 和 Parquet 文件中的数据。有关如何使用外部目录查询外部数据源的详细信息，请参阅[目录概述](../../../data_source/catalog/catalog_overview.md)以及与所需目录类型相关的主题。

## 语法

```Haskell
STRUCT<name, type>
```

- `name`：字段名称，与 CREATE TABLE 语句中定义的列名称相同。
- `type`：字段类型。它可以是任何受支持的类型。

## 在 StarRocks 中定义 STRUCT 列

您可以在创建表时定义 STRUCT 列，并将 STRUCT 数据加载到该列中。

```SQL
-- 定义一维结构。
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

-- 定义复杂结构。
CREATE TABLE t1(
  c0 INT,
  c1 STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>
)
DUPLICATE KEY(c0);

-- 定义 NOT NULL 结构。
CREATE TABLE t2(
  c0 INT,
  c1 STRUCT<a INT, b INT> NOT NULL
)
DUPLICATE KEY(c0);
```

具有 STRUCT 类型的列有以下限制：

- 不能用作表中的键列。它们只能用作值列。
- 不能用作表中的分区键列（在 PARTITION BY 之后）。
- 不能用作表中的分桶列（在 DISTRIBUTED BY 之后）。
- 仅当用作聚合表中的值列时才支持 replace() 函数。

## 在 SQL 中构造结构

STRUCT 可以使用以下函数在 SQL 中构造：[row、struct](../../sql-functions/struct-functions/row.md) 和 [named_struct](../../sql-functions/struct-functions/named_struct.md)。struct() 是 row() 的别名。

- `row` 和 `struct` 支持未命名结构。您无需指定字段名称。StarRocks 会自动生成列名，如 `col1`、`col2`...
- `named_struct` 支持命名结构。名称和值的表达式必须成对。

StarRocks 会根据输入值自动确定结构的类型。

```SQL
select row(1, 2, 3, 4) as numbers; -- 返回 {"col1":1,"col2":2,"col3":3,"col4":4}。
select row(1, 2, null, 4) as numbers; -- 返回 {"col1":1,"col2":2,"col3":null,"col4":4}。
select row(null) as nulls; -- 返回 {"col1":null}。
select struct(1, 2, 3, 4) as numbers; -- 返回 {"col1":1,"col2":2,"col3":3,"col4":4}。
select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4) as numbers; -- 返回 {"a":1,"b":2,"c":3,"d":4}。
```

## 加载 STRUCT 数据

您可以使用 [INSERT INTO](../../../loading/InsertInto.md) 和 [ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md) 两种方法将 STRUCT 数据加载到 StarRocks 中。

需要注意的是，StarRocks 会自动将数据类型转换为对应的 STRUCT 类型。

### INSERT INTO

```SQL
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

INSERT INTO t0 VALUES(1, row(1, 1));

SELECT * FROM t0;
+------+---------------+
| c0   | c1            |
+------+---------------+
|    1 | {"a":1,"b":1} |
+------+---------------+
```

### 从 ORC/Parquet 文件加载 STRUCT 数据

StarRocks 中的 STRUCT 数据类型对应 ORC 或 Parquet 格式的嵌套列结构。不需要额外的规格。您可以按照[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)中的说明从 ORC 或 Parquet 文件加载 STRUCT 数据。

## 访问 STRUCT 字段

要查询结构的子字段，可以使用点（`.`）运算符按字段名称查询值，也可以使用`[]`按索引调用值。

```Plain Text
mysql> select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4).a;
+------------------------------------------------+
| named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4).a |
+------------------------------------------------+
| 1                                              |
+------------------------------------------------+

mysql> select row(1, 2, 3, 4).col1;
+-----------------------+
| row(1, 2, 3, 4).col1  |
+-----------------------+
| 1                     |
+-----------------------+

mysql> select row(2, 4, 6, 8)[2];
+--------------------+
| row(2, 4, 6, 8)[2] |
+--------------------+
|                  4 |
+--------------------+

mysql> select row(map{'a':1}, 2, 3, 4)[1];
+-----------------------------+
| row(map{'a':1}, 2, 3, 4)[1] |
+-----------------------------+
| {"a":1}                     |
+-----------------------------+
```