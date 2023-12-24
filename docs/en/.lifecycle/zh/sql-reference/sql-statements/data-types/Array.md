---
displayed_sidebar: English
---

# 数组

ARRAY作为数据库的扩展类型，在PostgreSQL、ClickHouse、Snowflake等各种数据库系统中都得到支持。ARRAY被广泛应用于A/B测试、用户标签分析、用户画像等场景。StarRocks支持多维数组嵌套、数组切片、比较和过滤。

## 定义 ARRAY 列

您可以在创建表时定义ARRAY列。

~~~SQL
-- 定义一维数组。
ARRAY<type>

-- 定义嵌套数组。
ARRAY<ARRAY<type>>

-- 将数组列定义为NOT NULL。
ARRAY<type> NOT NULL
~~~

`type`指定数组中元素的数据类型。StarRocks支持以下元素类型：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、JSON、ARRAY（自v3.1起）、MAP（自v3.1起）和STRUCT（自v3.1起）。

数组中的元素默认可为空，例如`[null, 1 ,2]`。您不能将数组中的元素指定为NOT NULL。但是，您可以在创建表时将ARRAY列指定为NOT NULL，就像以下代码片段中的第三个示例一样。

示例：

~~~SQL
-- 将c1定义为元素类型为INT的一维数组。
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

-- 将c1定义为元素类型为VARCHAR的嵌套数组。
create table t1(
  c0 INT,
  c1 ARRAY<ARRAY<VARCHAR(10)>>
)
duplicate key(c0)
distributed by hash(c0);

-- 将c1定义为NOT NULL的数组列。
create table t2(
  c0 INT,
  c1 ARRAY<INT> NOT NULL
)
duplicate key(c0)
distributed by hash(c0);
~~~

## 限制

在StarRocks表中创建ARRAY列时，存在以下限制：

- 在v2.1之前的版本中，您只能在重复键表中创建ARRAY列。从v2.1开始，您还可以在其他类型的表（主键、唯一键、聚合）中创建ARRAY列。请注意，在Aggregate表中，仅当用于聚合ARRAY列中数据的函数为replace（）或replace_if_not_null（）时，才能创建该列。有关详细信息，请参阅[聚合表](../../../table_design/table_types/aggregate_table.md)。
- ARRAY列不能用作键列。
- ARRAY列不能用作分区键（包含在PARTITION BY中）或存储桶键（包含在DISTRIBUTED BY中）。
- ARRAY不支持DECIMAL V3。
- 一个数组最多可以有14级嵌套。

## 在 SQL 中构造数组

数组可以在SQL中使用方括号`[]`构造，每个数组元素用逗号`,`分隔。

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

如果数组由多种类型的元素组成，StarRocks会自动推断数据类型：

~~~Plain Text
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

NULL可以包含在元素中。

~~~Plain Text
mysql> select [1, NULL];

+----------+
| [1,NULL] |
+----------+
| [1,null] |
+----------+
~~~

对于空数组，可以使用尖括号来显示声明的类型，也可以直接写`[]`让StarRocks根据上下文推断类型。如果StarRocks无法推断类型，会报错。

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

## 加载数组数据

StarRocks支持通过三种方式加载Array数据：

- INSERT INTO适用于加载小规模数据进行测试。
- Broker Load适用于加载具有大规模数据的ORC或Parquet文件。
- Stream Load和Routine Load适用于加载包含大规模数据的CSV文件。

### 使用 INSERT INTO 加载数组

您可以使用INSERT INTO逐列加载小规模数据，或者在加载数据之前对数据进行ETL。

  ~~~SQL
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

INSERT INTO t0 VALUES(1, [1,2,3]);
~~~

### 使用 Broker Load 从 ORC 或 Parquet 文件加载数组

StarRocks中的数组类型与ORC和Parquet文件中的列表结构相对应，无需在StarRocks中指定不同的数据类型。有关数据加载的更多信息，请参阅[Broker load](../data-manipulation/BROKER_LOAD.md)。

### 使用 Stream Load 或 Routine Load 加载 CSV 格式的数组

默认情况下，CSV文件中的数组用逗号分隔。您可以在Kafka中使用[Stream Load](../../../loading/StreamLoad.md#load-csv-data)或[Routine Load](../../../loading/RoutineLoad.md#load-csv-format-data)来加载CSV文本文件或CSV数据。

## 查询ARRAY数据

您可以使用`[]`和下标访问数组中的元素，下标从`1`开始。

~~~Plain Text
mysql> select [1,2,3][1];

+------------+
| [1,2,3][1] |
+------------+
|          1 |
+------------+
1 row in set (0.00 sec)
~~~

如果下标为0或负数，**不会报告错误并返回NULL**。

~~~Plain Text
mysql> select [1,2,3][0];

+------------+
| [1,2,3][0] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
~~~

如果下标超过数组的长度（数组中的元素数），**将返回NULL**。

~~~Plain Text
mysql> select [1,2,3][4];

+------------+
| [1,2,3][4] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
~~~

对于多维数组，可以**递归访问元素**。

~~~Plain Text
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

