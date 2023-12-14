---
displayed_sidebar: "Chinese"
---

# 数组

数组作为数据库的扩展类型，受到多种数据库系统的支持，如PostgreSQL、ClickHouse和Snowflake。数组在A/B测试、用户标签分析和用户画像等场景中得到广泛应用。StarRocks支持多维数组嵌套、数组切片、比较和过滤。

## 定义数组列

您可以在创建表时定义数组列。

~~~SQL
-- 定义一维数组。
ARRAY<类型>

-- 定义嵌套数组。
ARRAY<ARRAY<类型>>

-- 将数组列定义为 NOT NULL。
ARRAY<类型> NOT NULL
~~~

`类型` 指定了数组中元素的数据类型。StarRocks支持以下元素类型：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、JSON、ARRAY（自v3.1起）、MAP（自v3.1起）和STRUCT（自v3.1起）。

数组中的元素默认可为空，例如`[null, 1, 2]`。您无法将数组中的元素指定为 NOT NULL。但是，在创建表时可以将数组列指定为 NOT NULL，例如以下代码片段中的第三个示例。

示例：

~~~SQL
-- 将 c1 定义为元素类型为 INT 的一维数组。
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

-- 将 c1 定义为元素类型为 VARCHAR 的嵌套数组。
create table t1(
  c0 INT,
  c1 ARRAY<ARRAY<VARCHAR(10)>>
)
duplicate key(c0)
distributed by hash(c0);

-- 将 c1 定义为 NOT NULL 数组列。
create table t2(
  c0 INT,
  c1 ARRAY<INT> NOT NULL
)
duplicate key(c0)
distributed by hash(c0);
~~~

## 限制

在创建StarRocks表的时候，针对数组列遵循以下限制：

- 在v2.1版本之前，只能在重复键表中创建数组列。从v2.1开始，还可以在其他类型的表（主键、唯一键、聚合）中创建数组列。请注意，在聚合表中，只能在对该列中的数据进行聚合的函数为 replace() 或 replace_if_not_null() 时，才能创建数组列。有关更多信息，请参见[聚合表](../../../table_design/table_types/aggregate_table.md)。
- 无法将数组列用作键列。
- 无法将数组列用作分区键（包括在PARTITION BY中）或分桶键（包括在DISTRIBUTED BY中）。
- 不支持DECIMAL V3在数组中的使用。
- 数组可以最多嵌套14层。

## 在SQL中构建数组

可以使用方括号`[]`在SQL中构建数组，每个数组元素之间用逗号`,`分隔。

~~~普通文本
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

如果数组包含多种类型的元素，StarRocks会自动推断数据类型：

~~~普通文本
mysql> select [1, 1.2] as floats;
+---------+
| floats  |
+---------+
| [1.0,1.2] |
+---------+

mysql> select [12, "100"];

+--------------+
| [12,'100']   |
+--------------+
| ["12","100"] |
+--------------+
~~~

您可以使用尖括号（`<>`）来显示声明的数组类型。

~~~普通文本
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

空值可以作为数组元素。

~~~普通文本
mysql> select [1, NULL];

+----------+
| [1,NULL] |
+----------+
| [1,null] |
+----------+
~~~

对于空数组，可以使用尖括号显示声明类型，或者可以直接写入 \[\]，让StarRocks根据上下文推断类型。如果StarRocks无法推断类型，就会报告错误。

~~~普通文本
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

## 加载数组数据

StarRocks支持以三种方式加载数组数据：

- INSERT INTO 适用于加载用于测试的小规模数据。
- Broker Load 适用于加载具有大规模数据的ORC或Parquet文件。
- Stream Load 和 Routine Load 适用于加载具有大规模数据的CSV文件。

### 使用 INSERT INTO 加载数组

您可以使用 INSERT INTO 逐列加载小规模数据，或者在加载数据前对数据进行ETL。

  ~~~SQL
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

INSERT INTO t0 VALUES(1, [1,2,3]);
~~~

### 使用 Broker Load 从ORC或Parquet文件加载数组

  StarRocks中的数组类型对应于ORC和Parquet文件中的列表结构，这消除了您需要在StarRocks中指定不同数据类型的需要。有关数据加载的更多信息，请参见[Broker load](../data-manipulation/BROKER_LOAD.md)。

### 使用 Stream Load 或 Routine Load 从CSV格式加载数组

  CSV文件中的数组默认使用逗号分隔。您可以使用[Stream Load](../../../loading/StreamLoad.md#load-csv-data)或[Routine Load](../../../loading/RoutineLoad.md#load-csv-format-data)加载CSV文本文件或Kafka中的CSV数据。

## 查询数组数据

您可以使用`[]`和下标访问数组中的元素，下标从`1`开始。

~~~普通文本
mysql> select [1,2,3][1];

+------------+
| [1,2,3][1] |
+------------+
|          1 |
+------------+
1 row in set (0.00 sec)
~~~

如果下标为 0 或负数，**不会报错并返回NULL**。

~~~普通文本
mysql> select [1,2,3][0];

+------------+
| [1,2,3][0] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
~~~

如果下标超出数组的长度（数组中元素的数量），**将返回NULL**。

~~~普通文本
mysql> select [1,2,3][4];

+------------+
| [1,2,3][4] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
~~~

对于多维数组，可以**递归**访问元素。

~~~普通文本
mysql(ARRAY)> select [[1,2],[3,4]][2];

+------------------+
| [[1,2],[3,4]][2] |
+------------------+
| [3,4]            |
+------------------+
1 row in set (0.00 sec)

mysql> select [[1,2],[3,4]][2][1];

+---------------------+
| [[1,2],[3,4]][2][1] |
+---------------------+
|                   3 |
+---------------------+
1 row in set (0.01 sec)
~~~