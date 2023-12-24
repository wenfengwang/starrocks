---
displayed_sidebar: English
---

# 创建表并选择

## 描述

您可以使用CREATE TABLE AS SELECT（CTAS）语句同步或异步查询表，并根据查询结果创建新表，然后将查询结果插入到新表中。

您可以使用[提交任务](../data-manipulation/SUBMIT_TASK.md)来提交异步CTAS任务。

## 语法

- 同步查询表，并根据查询结果创建新表，然后将查询结果插入到新表中。

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

- 异步查询表，并根据查询结果创建新表，然后将查询结果插入到新表中。

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

| **参数**     | **必填** | **描述**                                              |
| ----------------- | ------------ | ------------------------------------------------------------ |
| column_name       | 是          | 新表中的列名。您无需为列指定数据类型。StarRocks会自动为列指定合适的数据类型。StarRocks将FLOAT和DOUBLE数据转换为DECIMAL（38,9）数据。StarRocks还将CHAR、VARCHAR和STRING数据转换为VARCHAR（65533）数据。 |
| key_desc          | 否           | 语法为 `key_type ( <col_name1> [, <col_name2> , ...])`。<br />**参数**：<ul><li>`key_type`：[新表的键类型](../../../table_design/table_types/table_types.md)。有效值： `DUPLICATE KEY`和`PRIMARY KEY`。默认值：`DUPLICATE KEY`。</li><li> `col_name`：构成键的列。</li></ul> |
| COMMENT           | 否           | 新表的注释。                                |
| partition_desc    | 否           | 新表的分区方法。如果不指定此参数，默认情况下，新表没有分区。有关分区的详细信息，请参阅CREATE TABLE。 |
| distribution_desc | 否           | 新表的分桶方法。如果不指定此参数，存储桶列默认为成本优化器（CBO）收集的基数最高的列。存储桶数量默认为10。如果CBO未收集有关基数的信息，则存储桶列默认为新表中的第一列。有关分桶的详细信息，请参阅CREATE TABLE。 |
| Properties        | 否           | 新表的属性。                             |
| AS SELECT query   | 是          | 查询结果。 您可以在`... AS SELECT query`中指定列，例如`... AS SELECT a, b, c FROM table_a;`。在此示例中，`a`、`b`和`c`表示查询的表的列名。如果未指定新表的列名，则新表的列名也是`a`、`b`和`c`。您可以在`... AS SELECT query`中指定表达式，例如`... AS SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a;`。在此示例中，`a+1`、`b+2`表示查询表的列名，`c*c`、`x`、`y`、`z`注意：新表中的列数需要与SELECT语句中指定的列数相同。我们建议您使用易于识别的列名。|

## 使用说明

- CTAS语句只能创建满足以下要求的新表：
  - `ENGINE`为`OLAP`。

  - 默认情况下，表是Duplicate Key表。也可以在`key_desc`中将其指定为主键表。

  - 排序键为前三列，这三列数据类型的存储空间不超过36个字节。

- CTAS语句不支持为新创建的表设置索引。

- 如果由于FE重启等原因导致CTAS语句无法执行，可能会出现以下问题之一：
  - 新表已成功创建，但不包含数据。

  - 创建新表失败。

- 新表创建成功后，如果使用多个方法（如INSERT INTO）在新表中插入数据，则首先完成INSERT操作的方法会将其数据插入到新表中。

- 新表创建成功后，需要手动为用户授予该表的权限。

- 如果您异步查询表时没有指定任务名称，并根据查询结果创建新表，StarRocks会自动为该任务生成名称。

## 例子

示例1：同步查询表`order`，并根据查询结果创建新表`order_new`，然后将查询结果插入到新表中。

```SQL
CREATE TABLE order_new
AS SELECT * FROM order;
```

示例2：同步查询表中的`k1`、`k2`、`k3`列，并根据查询结果创建新表`order_new`，然后将查询结果插入到新表中。此外，将新表的列名设置为`a`、`b`和`c`。

```SQL
CREATE TABLE order_new (a, b, c)
AS SELECT k1, k2, k3 FROM order;
```

或

```SQL
CREATE TABLE order_new
AS SELECT k1 AS a, k2 AS b, k3 AS c FROM order;
```

示例3：同步查询表`employee`中`salary`列的最大值，并根据查询结果创建新表`employee_new`，然后将查询结果插入到新表中。此外，将新表的列名设置为`salary_max`。

```SQL
CREATE TABLE employee_new
AS SELECT MAX(salary) AS salary_max FROM employee;
```

插入数据后，查询新表。

```SQL
SELECT * FROM employee_new;

+------------+
| salary_max |
+------------+
|   10000    |
+------------+
```

示例4：使用CTAS创建主键表。请注意，主键表中的数据行数可能少于查询结果中的数据行数。这是因为主键表仅存储具有相同主键的一组行中的最新数据行。

```SQL
CREATE TABLE employee_new
PRIMARY KEY(order_id)
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

示例5：同步查询`lineorder`、`customer`、`supplier`、`part` 4个表，并根据查询结果创建新表`lineorder_flat`，然后将查询结果插入到新表中。此外，指定新表的分区方法和分桶方法。

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

示例6：异步查询表`order_detail`，并根据查询结果创建新表`order_statistics`，然后将查询结果插入到新表中。

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

-- 任务信息

TASK_NAME: ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2
CREATE_TIME: 2022-06-14 14:07:06
SCHEDULE: MANUAL
DATABASE: default_cluster:test
DEFINITION: CREATE TABLE order_statistics AS SELECT COUNT(*) as cnt FROM order_detail
EXPIRE_TIME: 2022-06-17 14:07:06
```

检查任务运行的状态。

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;

-- 任务运行的状态

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

当任务运行的状态为`SUCCESS`时，查询新表。

```SQL
SELECT * FROM order_statistics;
```
