---
displayed_sidebar: English
---

# 创建表格作为选择

## 描述

您可以使用 CREATE TABLE AS SELECT (CTAS) 语句来同步或异步地查询一个表，并基于查询结果来创建一个新表，然后将查询结果插入新表中。

您可以使用 [SUBMIT TASK](../data-manipulation/SUBMIT_TASK.md) 提交一个异步的 CTAS 任务。

## 语法

- 同步地查询一张表，并基于查询结果创建一个新表，然后将查询结果插入到新表中。

  ```SQL
  CREATE TABLE [IF NOT EXISTS] [database.]table_name
  [(column_name [, column_name2, ...]]
  [key_desc]
  [COMMENT "table comment"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)]
  AS SELECT query
  [ ... ]
  ```

- 异步地查询一张表，并基于查询结果创建一个新表，然后将查询结果插入到新表中。

  ```SQL
  SUBMIT [/*+ SET_VAR(key=value) */] TASK [[database.]<task_name>]AS
  CREATE TABLE [IF NOT EXISTS] [database.]table_name
  [(column_name [, column_name2, ...]]
  [key_desc]
  [COMMENT "table comment"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)]AS SELECT query
  [ ... ]
  ```

## 参数

|参数|必填|说明|
|---|---|---|
|column_name|是|新表中的列的名称。您不需要指定列的数据类型。 StarRocks 自动为列指定适当的数据类型。 StarRocks 将 FLOAT 和 DOUBLE 数据转换为 DECIMAL(38,9) 数据。 StarRocks 还将 CHAR、VARCHAR 和 STRING 数据转换为 VARCHAR(65533) 数据。|
|key_desc|否|语法为 key_type ( <col_name1> [, <col_name2> , ...])。参数：key_type：新表的键类型。有效值：重复键和主键。默认值：重复密钥。 col_name：构成键的列。|
|COMMENT|否|新表的注释。|
|partition_desc|否|新表的分区方式。如果不指定该参数，则默认新表没有分区。有关分区的详细信息，请参阅创建表。|
|distribution_desc|否|新表的分桶方法。如果不指定此参数，则存储桶列默认为基于成本的优化器 (CBO) 收集的基数最高的列。桶的数量默认为10。如果CBO不收集有关基数的信息，则桶列默认为新表中的第一列。有关分桶的更多信息，请参阅创建表。|
|属性|否|新表的属性。|
|AS SELECT 查询|是|查询结果。您可以在... AS SELECT 查询中指定列，例如... AS SELECT a, b, c FROM table_a;。本例中a、b、c表示查询的表的列名。如果不指定新表的列名，则新表的列名也为a、b、c。您可以在... AS SELECT 查询中指定表达式，例如... AS SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a;。本例中，a+1、b+2、c*c表示查询的表的列名，x、y、z表示新表的列名。注意：新表的列数需要与SELECT语句中指定的列数相同。我们建议您使用易于识别的列名称。|

## 使用须知

- CTAS 语句只能创建满足以下条件的新表：
-   引擎类型为 OLAP。

-   默认情况下，表是一个 Duplicate Key 表。您也可以在 key_desc 中将其指定为 Primary Key 表。

-   排序键为前三列，且这三列数据类型的存储空间总和不超过 36 字节。

- CTAS 语句不支持为新创建的表设置索引。

- 如果由于 FE 重启等原因导致 CTAS 语句执行失败，可能会出现以下问题：
-   新表创建成功，但不包含数据。

-   新表创建失败。

- 新表创建后，如果使用多种方法（例如 INSERT INTO）向新表插入数据，那么首个完成插入操作的方法将会将其数据插入到新表中。

- 新表创建后，您需要手动为用户授权此表的权限。

- 如果您在异步查询表并基于查询结果创建新表时没有指定任务名称，StarRocks 将自动为该任务生成一个名称。

## 示例

示例 1：同步查询 order 表，并基于查询结果创建新表 order_new，然后将查询结果插入到新表中。

```SQL
CREATE TABLE order_new
AS SELECT * FROM order;
```

示例 2：同步查询 order 表中的 k1、k2、k3 列，并基于查询结果创建新表 order_new，接着将查询结果插入到新表中。同时，将新表的列名设置为 a、b、c。

```SQL
CREATE TABLE order_new (a, b, c)
AS SELECT k1, k2, k3 FROM order;
```

或者

```SQL
CREATE TABLE order_new
AS SELECT k1 AS a, k2 AS b, k3 AS c FROM order;
```

示例 3：同步查询 employee 表中 salary 列的最大值，并基于查询结果创建新表 employee_new，然后将查询结果插入到新表中。同时，将新表的列名设置为 salary_max。

```SQL
CREATE TABLE employee_new
AS SELECT MAX(salary) AS salary_max FROM employee;
```

数据插入后，查询新表。

```SQL
SELECT * FROM employee_new;

+------------+
| salary_max |
+------------+
|   10000    |
+------------+
```

示例 4：使用 CTAS 创建一个 Primary Key 表。请注意，Primary Key 表中的数据行数可能少于查询结果中的数据行数。这是因为[Primary Key](../../../table_design/table_types/primary_key_table.md)表只存储具有相同主键的一组行中最新的数据行。

```SQL
CREATE TABLE employee_new
PRIMARY KEY(order_id)
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

示例 5：同步查询 lineorder、customer、supplier、part 四张表，并基于查询结果创建新表 lineorder_flat，然后将查询结果插入到新表中。同时，指定新表的分区方法和桶分布方法。

```SQL
CREATE TABLE lineorder_flat
PARTITION BY RANGE(`LO_ORDERDATE`)
(
    START ("1993-01-01") END ("1999-01-01") EVERY (INTERVAL 1 YEAR)
)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) AS SELECT
    l.LO_ORDERKEY AS LO_ORDERKEY,
    l.LO_LINENUMBER AS LO_LINENUMBER,
    l.LO_CUSTKEY AS LO_CUSTKEY,
    l.LO_PARTKEY AS LO_PARTKEY,
    l.LO_SUPPKEY AS LO_SUPPKEY,
    l.LO_ORDERDATE AS LO_ORDERDATE,
    l.LO_ORDERPRIORITY AS LO_ORDERPRIORITY,
    l.LO_SHIPPRIORITY AS LO_SHIPPRIORITY,
    l.LO_QUANTITY AS LO_QUANTITY,
    l.LO_EXTENDEDPRICE AS LO_EXTENDEDPRICE,
    l.LO_ORDTOTALPRICE AS LO_ORDTOTALPRICE,
    l.LO_DISCOUNT AS LO_DISCOUNT,
    l.LO_REVENUE AS LO_REVENUE,
    l.LO_SUPPLYCOST AS LO_SUPPLYCOST,
    l.LO_TAX AS LO_TAX,
    l.LO_COMMITDATE AS LO_COMMITDATE,
    l.LO_SHIPMODE AS LO_SHIPMODE,
    c.C_NAME AS C_NAME,
    c.C_ADDRESS AS C_ADDRESS,
    c.C_CITY AS C_CITY,
    c.C_NATION AS C_NATION,
    c.C_REGION AS C_REGION,
    c.C_PHONE AS C_PHONE,
    c.C_MKTSEGMENT AS C_MKTSEGMENT,
    s.S_NAME AS S_NAME,
    s.S_ADDRESS AS S_ADDRESS,
    s.S_CITY AS S_CITY,
    s.S_NATION AS S_NATION,
    s.S_REGION AS S_REGION,
    s.S_PHONE AS S_PHONE,
    p.P_NAME AS P_NAME,
    p.P_MFGR AS P_MFGR,
    p.P_CATEGORY AS P_CATEGORY,
    p.P_BRAND AS P_BRAND,
    p.P_COLOR AS P_COLOR,
    p.P_TYPE AS P_TYPE,
    p.P_SIZE AS P_SIZE,
    p.P_CONTAINER AS P_CONTAINER FROM lineorder AS l
INNER JOIN customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
INNER JOIN supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
INNER JOIN part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```

示例 6：异步查询 order_detail 表，并基于查询结果创建新表 order_statistics，然后将查询结果插入到新表中。

```plaintext
SUBMIT TASK AS CREATE TABLE order_statistics AS SELECT COUNT(*) as count FROM order_detail;

+-------------------------------------------+-----------+
| TaskName                                  | Status    |
+-------------------------------------------+-----------+
| ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2 | SUBMITTED |
+-------------------------------------------+-----------+
```

检查任务信息。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;

-- Information of the Task

TASK_NAME: ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2
CREATE_TIME: 2022-06-14 14:07:06
SCHEDULE: MANUAL
DATABASE: default_cluster:test
DEFINITION: CREATE TABLE order_statistics AS SELECT COUNT(*) as cnt FROM order_detail
EXPIRE_TIME: 2022-06-17 14:07:06
```

检查 TaskRun 的状态。

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;

-- State of the TaskRun

QUERY_ID: 37bd2b63-eba8-11ec-8d41-bab8ee315bf2
TASK_NAME: ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2
CREATE_TIME: 2022-06-14 14:07:06
FINISH_TIME: 2022-06-14 14:07:07
STATE: SUCCESS
DATABASE: 
DEFINITION: CREATE TABLE order_statistics AS SELECT COUNT(*) as cnt FROM order_detail
EXPIRE_TIME: 2022-06-17 14:07:06
ERROR_CODE: 0
ERROR_MESSAGE: NULL
```

当 TaskRun 的状态为 SUCCESS 时，查询新表。

```SQL
SELECT * FROM order_statistics;
```
