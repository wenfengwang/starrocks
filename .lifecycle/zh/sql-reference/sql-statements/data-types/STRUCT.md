---
displayed_sidebar: English
---

# 结构体（STRUCT）

## 描述

STRUCT 是用来表达复杂数据类型的常用数据结构。它代表了一个元素集合（也称为字段），这些元素具有不同的数据类型，例如 `<a INT, b STRING>`。

在一个结构体中，字段名称必须是唯一的。字段可以是基本数据类型（如数值、字符串或日期）或更复杂的数据类型（如 ARRAY 或 MAP）。

结构体的字段还可以是另一个 STRUCT、ARRAY 或 MAP，这使得可以创建嵌套的数据结构，例如 `STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>`。

STRUCT 数据类型从 v3.1 版本开始得到支持。在 v3.1 版本中，您可以在创建 StarRocks 表时定义 STRUCT 类型的列，将 STRUCT 数据加载到该表中，并查询 MAP 数据。

从 v2.5 版本开始，StarRocks支持查询复杂数据类型**MAP**和**STRUCT**。您可以利用StarRocks提供的外部目录功能，从Apache Hive™、Apache Hudi和Apache Iceberg中查询**MAP**和**STRUCT**数据。目前仅支持查询**ORC**和**Parquet**文件格式的数据。有关如何使用外部目录来查询外部数据源的更多信息，请参见[Overview of catalogs](../../../data_source/catalog/catalog_overview.md)以及相关的目录类型所需的主题。

## 语法

```Haskell
STRUCT<name, type>
```

- name：字段名，与 CREATE TABLE 语句中定义的列名相同。
- type：字段类型，可以是任何支持的数据类型。

## 在 StarRocks 中定义 STRUCT 列

您可以在创建表时定义 STRUCT 类型的列，并将 STRUCT 数据加载到这些列中。

```SQL
-- Define a one-dimensional struct.
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

-- Define a complex struct.
CREATE TABLE t1(
  c0 INT,
  c1 STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>
)
DUPLICATE KEY(c0);

-- Define a NOT NULL struct.
CREATE TABLE t2(
  c0 INT,
  c1 STRUCT<a INT, b INT> NOT NULL
)
DUPLICATE KEY(c0);
```

具有 STRUCT 类型的列有以下限制：

- 不能作为表中的键列使用，它们只能作为值列。
- 不能作为表中的分区键列（跟在 PARTITION BY 之后）。
- 不能作为表中的分桶列（跟在 DISTRIBUTED BY 之后）。
- 只在作为值列时支持`replace()`函数，当用作[聚合表](../../../table_design/table_types/aggregate_table.md)中的值列时。

## 在 SQL 中构建 STRUCT

STRUCT 可以使用以下函数在 SQL 中构建：[row, struct](../../sql-functions/struct-functions/row.md)，和 [named_struct](../../sql-functions/struct-functions/named_struct.md)。struct() 是 row() 的别名。

- row 和 struct 支持未命名的结构体。您无需指定字段名称，StarRocks 会自动生成列名，如 col1、col2 等。
- named_struct 支持命名的结构体。字段名称和值的表达式必须成对出现。

StarRocks 会根据输入值自动确定 STRUCT 的类型。

```SQL
select row(1, 2, 3, 4) as numbers; -- Return {"col1":1,"col2":2,"col3":3,"col4":4}.
select row(1, 2, null, 4) as numbers; -- Return {"col1":1,"col2":2,"col3":null,"col4":4}.
select row(null) as nulls; -- Return {"col1":null}.
select struct(1, 2, 3, 4) as numbers; -- Return {"col1":1,"col2":2,"col3":3,"col4":4}.
select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4) as numbers; -- Return {"a":1,"b":2,"c":3,"d":4}.
```

## 加载 STRUCT 数据

您可以通过两种方法将 STRUCT 数据加载到 StarRocks 中：[INSERT INTO](../../../loading/InsertInto.md)，和[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)。

请注意，StarRocks 会自动将数据类型转换成对应的 STRUCT 类型。

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

StarRocks 中的 **STRUCT** 数据类型与 ORC 或 Parquet 格式中的嵌套列结构相对应，无需额外的配置即可。您可以遵循 [ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md) 的指南，从 ORC 或 Parquet 文件中加载 **STRUCT** 数据。

## 访问 STRUCT 字段

要查询结构体的子字段，您可以使用点（.）运算符通过字段名称查询值，或使用方括号（[]）通过索引访问值。

```Plain
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
