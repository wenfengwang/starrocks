---
displayed_sidebar: English
---

# ARRAY

ARRAY 作为数据库的扩展类型，在 PostgreSQL、ClickHouse、Snowflake 等多种数据库系统中都得到支持。ARRAY 广泛应用于 A/B 测试、用户标签分析、用户画像等场景。StarRocks 支持多维数组嵌套、数组切片、比较和过滤。

## 定义 ARRAY 列

您可以在创建表时定义 ARRAY 列。

```SQL
-- 定义一维数组。
ARRAY<type>

-- 定义嵌套数组。
ARRAY<ARRAY<type>>

-- 定义一个 NOT NULL 的数组列。
ARRAY<type> NOT NULL
```

`type` 指定数组中元素的数据类型。StarRocks 支持以下元素类型：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、JSON、ARRAY（自 v3.1 起）、MAP（自 v3.1 起）和 STRUCT（自 v3.1 起）。

数组中的元素默认可以为 null，例如 `[null, 1, 2]`。不能将数组中的元素指定为 NOT NULL。但是，您可以在创建表时将 ARRAY 列指定为 NOT NULL，如以下代码片段中的第三个示例所示。

示例：

```SQL
-- 定义 c1 为元素类型为 INT 的一维数组。
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

-- 定义 c1 为元素类型为 VARCHAR 的嵌套数组。
create table t1(
  c0 INT,
  c1 ARRAY<ARRAY<VARCHAR(10)>>
)
duplicate key(c0)
distributed by hash(c0);

-- 定义 c1 为 NOT NULL 的数组列。
create table t2(
  c0 INT,
  c1 ARRAY<INT> NOT NULL
)
duplicate key(c0)
distributed by hash(c0);
```

## 限制

在 StarRocks 表中创建 ARRAY 列时，适用以下限制：

- 在 v2.1 之前的版本中，只能在 Duplicate Key 表中创建 ARRAY 列。从 v2.1 开始，您也可以在其他类型的表（Primary Key、Unique Key、Aggregate）中创建 ARRAY 列。请注意，在 Aggregate 表中，只有当用于聚合该列中数据的函数是 replace() 或 replace_if_not_null() 时，才能创建 ARRAY 列。更多信息，请参见 [聚合表](../../../table_design/table_types/aggregate_table.md)。
- ARRAY 列不能用作键列。
- ARRAY 列不能用作分区键（包含在 PARTITION BY 中）或分桶键（包含在 DISTRIBUTED BY 中）。
- ARRAY 不支持 DECIMAL V3 类型。
- 数组最多可以有 14 级嵌套。

## 在 SQL 中构造数组

可以使用方括号 `[]` 在 SQL 中构造数组，每个数组元素之间用逗号 `,` 分隔。

```Plain
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
```

如果数组由多种类型的元素组成，StarRocks 会自动推断数据类型：

```Plain
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
```

您可以使用尖括号 `<>` 来指定声明的数组类型。

```Plain
mysql> select ARRAY<float>[1, 2];

+-----------------------+
| ARRAY<float>[1.0,2.0] |
+-----------------------+
| [1,2]                 |
+-----------------------+

mysql> select ARRAY<INT>["12", "100"];

+------------------------+
| ARRAY<int>[12,100]     |
+------------------------+
| [12,100]               |
+------------------------+
```

元素中可以包含 NULL。

```Plain
mysql> select [1, NULL];

+----------+
| [1,NULL] |
+----------+
| [1,null] |
+----------+
```

对于空数组，您可以使用尖括号来指定声明的类型，或者直接写 `[]` 让 StarRocks 根据上下文推断类型。如果 StarRocks 无法推断类型，将报告错误。

```Plain
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
```

## 加载数组数据

StarRocks 支持三种方式加载 Array 数据：

- INSERT INTO 适合加载小规模数据进行测试。
- Broker Load 适合加载具有大规模数据的 ORC 或 Parquet 文件。
- Stream Load 和 Routine Load 适合加载大规模数据的 CSV 文件。

### 使用 INSERT INTO 加载数组

您可以使用 INSERT INTO 逐列加载小规模数据，或者在加载数据之前对数据进行 ETL。

```SQL
create table t0(
c0 INT,
c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

INSERT INTO t0 VALUES(1, [1,2,3]);
```

### 使用 Broker Load 从 ORC 或 Parquet 文件加载数组

StarRocks 中的数组类型对应于 ORC 和 Parquet 文件中的 list 结构，这意味着您无需在 StarRocks 中指定不同的数据类型。有关数据加载的更多信息，请参阅 [Broker Load](../data-manipulation/BROKER_LOAD.md)。

### 使用 Stream Load 或 Routine Load 加载 CSV 格式的数组

CSV 文件中的数组默认以逗号分隔。您可以使用 [Stream Load](../../../loading/StreamLoad.md#load-csv-data) 或 [Routine Load](../../../loading/RoutineLoad.md#load-csv-format-data) 加载 Kafka 中的 CSV 文本文件或 CSV 数据。

## 查询 ARRAY 数据

您可以使用 `[]` 和下标访问数组中的元素，下标从 `1` 开始。

```Plain
mysql> select [1,2,3][1];

+------------+
| [1,2,3][1] |
+------------+
|          1 |
+------------+
1 row in set (0.00 sec)
```

如果下标为 0 或负数，**不会报错，返回 NULL**。

```Plain
mysql> select [1,2,3][0];

+------------+
| [1,2,3][0] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
```

如果下标超过数组长度（数组中的元素数量），**将返回 NULL**。

```Plain
mysql> select [1,2,3][4];

+------------+
| [1,2,3][4] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
```

对于多维数组，可以**递归地**访问元素。

```Plain
mysql> select [[1,2],[3,4]][2];

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
```