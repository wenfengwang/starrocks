---
displayed_sidebar: "中文"
---

# 数组

本文介绍如何在 StarRocks 中使用数组类型。

数组（Array）是数据库中的一种扩展数据类型，它能够在多种数据库系统中得到支持，广泛应用于 A/B 测试对比、用户标签分析、人群画像等场景。StarRocks 当前支持多维数组嵌套、数组切片、比较、过滤等功能。

## 定义数组类型的列

您可以在创建表时定义数组类型的列。

~~~SQL
-- 定义一维数组。
ARRAY<type>

-- 定义嵌套数组。
ARRAY<ARRAY<type>>

-- 定义 NOT NULL 数组列。
ARRAY<type> NOT NULL
~~~

数组列的定义格式为 `ARRAY<type>`，其中 `type` 指代数组内元素的类型。目前支持的数组元素类型包括：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、JSON、BINARY（3.0 及以后版本）、MAP（3.1 及以后版本）、STRUCT（3.1 及以后版本）、Fast Decimal（3.1 及以后版本）。

数组内的元素默认可以为 NULL，例如 [NULL, 1, 2]。暂不支持指定数组元素为非 NULL，但可以定义数组列本身为非 NULL，如上面的第三个示例所示。

> **注意**
>
> 使用数组类型列时有以下限制：
>
> * StarRocks 2.1 版本之前，只支持在明细模型表（Duplicate Key）中定义数组类型列。2.1 版本起，支持在 Primary Key、Unique Key、Aggregate Key 模型表中定义数组类型列。但在聚合模型表（Aggregate Key）中，仅当聚合列的聚合函数为 replace 或 replace_if_not_null 时，才支持将该列定义为数组类型。
> * 数组列暂时不能作为 Key 列。
> * 数组列不能作为分桶（Distributed By）列。
> * 数组列不能作为分区（Partition By）列。
> * 数组列目前不支持 DECIMAL V3 数据类型。
> * 数组列最多支持 14 层嵌套。

示例：

~~~SQL
-- 创建表并指定 `c1` 列为 INT 类型的一维数组。
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

-- 创建表并指定 `c1` 为 VARCHAR 类型的两层嵌套数组。
create table t1(
  c0 INT,
  c1 ARRAY<ARRAY<VARCHAR(10)>>
)
duplicate key(c0)
distributed by hash(c0);

-- 创建表并定义非 NULL 的数组列。
create table t2(
  c0 INT,
  c1 ARRAY<INT> NOT NULL
)
duplicate key(c0)
distributed by hash(c0);
~~~

## 使用 SELECT 语句构造数组

您可以在 SQL 语句中通过中括号（`[]`）构造数组常量，数组元素之间用逗号（`,`）分隔。

示例：

~~~Plain Text
mysql> select [1, 2, 3] as numbers;

+---------+
| numbers |
+---------+
| [1,2,3] |
+---------+

mysql> select ["apple", "orange", "pear"] as fruit;

+---------------------------+
| fruit                     |
+---------------------------+
| ["apple","orange","pear"] |
+---------------------------+

mysql> select [true, false] as booleans;

+----------+
| booleans |
+----------+
| [1,0]    |
+----------+
~~~

数组元素中含有不同类型时，StarRocks 将自动推导出合适的类型（supertype）。

~~~Plain Text
mysql> select [1, 1.2] as floats;

+---------+
| floats  |
+---------+
| [1,1.2] |
+---------+

mysql> select [12, "100"];

+--------------+
| [12,'100']   |
+--------------+
| ["12","100"] |
+--------------+
~~~

您可以使用尖括号（`<>`）指明数组类型。

~~~Plain Text
mysql> select ARRAY<float>[1, 2];

+-----------------------+
| ARRAY<float>[1.0,2.0] |
+-----------------------+
| [1,2]                 |
+-----------------------+

mysql> select ARRAY<INT>["12", "100"];

+------------------------+
| ARRAY<int(11)>[12,100] |
+------------------------+
| [12,100]               |
+------------------------+
~~~

数组元素中也可以包含 NULL。

~~~Plain Text
mysql> select [1, NULL];

+----------+
| [1,NULL] |
+----------+
| [1,null] |
+----------+
~~~

定义空数组时，可以用尖括号指定其类型。您也可以直接定义为 `[]`，此时 StarRocks 会根据上下文推断其类型，如果无法推断则报错。

~~~Plain Text
mysql> select [];

+------+
| []   |
+------+
| []   |
+------+

mysql> select ARRAY<VARCHAR(10)>[];

+----------------------------------+
| ARRAY<unknown type: NULL_TYPE>[] |
+----------------------------------+
| []                               |
+----------------------------------+

mysql> select array_append([], 10);

+----------------------+
| array_append([], 10) |
+----------------------+
| [10]                 |
+----------------------+
~~~

## 导入数组类型的数据

StarRocks 当前支持三种方式来写入数组数据。

### 通过 INSERT INTO 语句导入数组

INSERT INTO 语句导入适合小批量数据逐行导入，以及对 StarRocks 内外部表数据进行 ETL 处理并导入。

示例：

~~~SQL
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);
INSERT INTO t0 VALUES(1, [1,2,3]);
~~~

### 通过 Broker Load 批量导入 ORC 或 Parquet 文件中的数组

StarRocks 的数组类型与 ORC 或 Parquet 格式中的 List 结构相对应，无需进行特别指定。具体导入方法请参见 [Broker Load](../data-manipulation/BROKER_LOAD.md)。

当前 StarRocks 支持导入 ORC 文件中的 List 结构。Parquet 格式导入功能正在开发中。

### 通过 Stream Load 或 Routine Load 导入 CSV 格式数组

您可以使用 [Stream Load](../../../loading/StreamLoad.md#导入-csv-格式的数据) 或 [Routine Load](../../../loading/RoutineLoad.md#导入-csv-数据) 方法导入 CSV 文本文件或 Kafka 中的 CSV 格式数据，默认采用逗号分隔。

## 访问数组中的元素

您可以使用中括号（`[]`）和下标访问数组中的某个元素。下标从 `1` 开始。

~~~Plain Text
mysql> select [1,2,3][1];

+------------+
| [1,2,3][1] |
+------------+
|          1 |
+------------+
~~~

如果下标为 `0` 或负数，StarRocks **不会报错，但会返回 NULL**。

~~~Plain Text
mysql> select [1,2,3][0];

+------------+
| [1,2,3][0] |
+------------+
|       NULL |
+------------+
~~~

如果下标超过数组大小，StarRocks **也会返回 NULL**。

~~~Plain Text
mysql> select [1,2,3][4];

+------------+
| [1,2,3][4] |
+------------+
|       NULL |
+------------+
~~~

要访问多维数组中的元素，您可以通过**递归**的方式。

~~~Plain Text
mysql> select [[1,2],[3,4]][2];

+------------------+
| [[1,2],[3,4]][2] |
```
+------------------+
| [3,4]            |
+------------------+

mysql> select [[1,2],[3,4]][2][1];

+---------------------+
| [[1,2],[3,4]][2][1] |
+---------------------+
|                   3 |
+---------------------+
~~~