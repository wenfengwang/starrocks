---
displayed_sidebar: "Chinese"
---

# 结构体

## 描述

STRUCT广泛用于表示复杂的数据类型。它代表了具有不同数据类型的元素（也称为字段）的集合，例如`<a INT, b STRING>`。

结构体中的字段名必顶是唯一的。字段可以是基本数据类型（如数字、字符串或日期）或复杂数据类型（如数组或映射）。

结构体中的字段也可以是另一个结构体、数组或映射，这使你能够创建嵌套数据结构，例如`STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>`。

STRUCT数据类型从v3.1开始得到支持。在v3.1中，您可以在创建StarRocks表时定义STRUCT列，将STRUCT数据加载到该表中，并查询映射数据。

从v2.5开始，StarRocks支持从数据湖查询复杂数据类型MAP和STRUCT。您可以使用StarRocks提供的外部目录来查询来自Apache Hive™、Apache Hudi和Apache Iceberg的MAP和STRUCT数据。您只能查询ORC和Parquet文件中的数据。有关如何使用外部目录查询外部数据源的详细信息，请参见[目录概览](../../../data_source/catalog/catalog_overview.md)及相关的目录类型的主题。

## 语法

```Haskell
STRUCT<字段名, 类型>
```

- `字段名`：字段名，与CREATE TABLE语句中定义的列名相同。
- `类型`：字段类型。可以是任何支持的类型。

## 在StarRocks中定义STRUCT列

您可以在创建表时定义STRUCT列，并将STRUCT数据加载到该列中。

```SQL
-- 定义一维结构体。
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

-- 定义复杂结构体。
CREATE TABLE t1(
  c0 INT,
  c1 STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>
)
DUPLICATE KEY(c0);

-- 定义非空结构体。
CREATE TABLE t2(
  c0 INT,
  c1 STRUCT<a INT, b INT> NOT NULL
)
DUPLICATE KEY(c0);
```

具有STRUCT类型的列具有以下限制：

- 不能在表中用作键列。它们只能用作值列。
- 不能在表中作为分区键列（接PARTITION BY）使用。
- 不能在表中用作分桶列（接DISTRIBUTED BY）。
- 在[聚合表](../../../table_design/table_types/aggregate_table.md)中只支持replace()函数用作值列。

## 在SQL中构造结构体

可以使用以下函数在SQL中构造STRUCT：[row，struct](../../sql-functions/struct-functions/row.md)和[named_struct](../../sql-functions/struct-functions/named_struct.md)。struct()是row()的别名。

- `row`和`struct`支持无名称的结构。您不需要指定字段名。StarRocks会自动生成列名，如`col1`，`col2`...
- `named_struct`支持有名称的结构。名称和值的表达式必须成对出现。

StarRocks会根据输入值自动确定结构的类型。

```SQL
select row(1, 2, 3, 4) as numbers; -- 返回{"col1":1,"col2":2,"col3":3,"col4":4}。
select row(1, 2, null, 4) as numbers; -- 返回{"col1":1,"col2":2,"col3":null,"col4":4}。
select row(null) as nulls; -- 返回{"col1":null}。
select struct(1, 2, 3, 4) as numbers; -- 返回{"col1":1,"col2":2,"col3":3,"col4":4}。
select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4) as numbers; -- 返回{"a":1,"b":2,"c":3,"d":4}。
```

## 加载STRUCT数据

可以使用两种方法将STRUCT数据加载到StarRocks中：[INSERT INTO](../../../loading/InsertInto.md)和[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)。

注意，StarRocks会自动将数据类型转换为相应的STRUCT类型。

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

### 从ORC/Parquet文件中加载STRUCT数据

StarRocks中的STRUCT数据类型对应于ORC或Parquet格式中的嵌套列结构。不需要额外的规范。您可以按照[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)中的说明从ORC或Parquet文件中加载STRUCT数据。

## 访问STRUCT字段

要查询结构体的子字段，您可以使用点(`.`)运算符按照字段名查询值，或使用`[]`按索引调用值。

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