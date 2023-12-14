---
displayed_sidebar: "Chinese"
---

# 在查询语句中创建表

## 描述

您可以使用CREATE TABLE AS SELECT（简称CTAS）语句来同步或异步查询表，并根据查询结果创建一个新表，然后将查询结果插入新表。

您可以使用[SUBMIT TASK](../data-manipulation/SUBMIT_TASK.md)来提交一个异步的CTAS任务。

## 语法

- 同步查询表并根据查询结果创建一个新表，然后将查询结果插入新表。

  ```SQL
  CREATE TABLE [IF NOT EXISTS] [数据库.]表名
  [(列名 [, 列名2, ...]]
  [key_desc]
  [COMMENT "表注释"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)]
  AS SELECT 查询
  [ ... ]
  ```

- 异步查询表并根据查询结果创建一个新表，然后将查询结果插入新表。

  ```SQL
  SUBMIT [/*+ SET_VAR(key=value) */] TASK [[数据库.]<任务名称>]AS
  CREATE TABLE [IF NOT EXISTS] [数据库.]表名
  [(列名 [, 列名2, ...]]
  [key_desc]
  [COMMENT "表注释"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)]AS SELECT 查询
  [ ... ]
  ```

## 参数

| **参数**          | **必需** | **描述**                                                    |
| ----------------- | ------------ | ------------------------------------------------------------ |
| 列名              | 是           | 新表中的列名。您无需为列指定数据类型。StarRocks会自动为列指定适当的数据类型。StarRocks会将FLOAT和DOUBLE数据转换为DECIMAL(38,9)数据。StarRocks还会将CHAR、VARCHAR和STRING数据转换为VARCHAR(65533)数据。 |
| key_desc          | 否           | 语法为`key_type ( <col_name1> [, <col_name2> , ...])`。<br />**参数**:<ul><li>`key_type`: [新表的键类型](../../../table_design/table_types/table_types.md)。有效值：`DUPLICATE KEY`和`PRIMARY KEY`。默认值：`DUPLICATE KEY`。</li><li> `col_name`: 构成键的列。</li></ul> |
| COMMENT           | 否           | 新表的注释。                                               |
| partition_desc    | 否           | 新表的分区方法。如果您不指定此参数，默认情况下，新表没有分区。有关分区的更多信息，请参见CREATE TABLE。 |
| distribution_desc | 否           | 新表的分桶方法。如果您不指定此参数，默认情况下，桶列默认为成本优化器（CBO）收集的具有最高基数的列。桶的数量默认为10。如果CBO未收集有关基数的信息，则桶列默认为新表中的第一列。有关分桶的更多信息，请参见CREATE TABLE。 |
| 属性            | 否           | 新表的属性。                                               |
| AS SELECT 查询   | 是          | 查询结果。您可以在`... AS SELECT 查询`中指定列，例如，`... AS SELECT a, b, c FROM table_a;`。在此示例中，`a`、`b`和`c`表示被查询表的列名。您可以在`... AS SELECT 查询`中指定表达式，例如，`... AS SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a;`。在此示例中，`a+1`、`b+2`和`c*c`表示被查询表的列名，`x`、`y`和`z`表示新表的列名。注意：新表中的列数需要和SELECT语句指定的列数相同。我们建议您使用便于识别的列名。 |

## 使用说明

- CTAS语句只能创建满足以下要求的新表：

  - `ENGINE`为`OLAP`。

  - 表默认为Duplicate Key表。您还可以在`key_desc`中指定为Primary Key表。

  - 排序键是前三列，并且这三列的数据类型的存储空间不超过36个字节。

- CTAS语句不支持给新创建的表设置索引。

- 如果CTAS语句由于FE重新启动等原因执行失败，可能出现以下问题之一：

  - 成功创建新表但不包含数据。

  - 无法创建新表。

- 新表创建后，如果使用多种方法（例如INSERT INTO）向新表插入数据，最先完成INSERT操作的方法将把数据插入新表中。

- 新表创建后，您需要手动为用户授予权限。

- 如果您在异步查询表并根据查询结果创建新表时未指定任务名称，StarRocks会自动生成任务名称。

## 示例

示例1：同步查询表`order`并根据查询结果创建新表`order_new`，然后将查询结果插入新表。

```SQL
CREATE TABLE order_new
AS SELECT * FROM order;
```

示例2：同步查询表`order`中的`k1`、`k2`和`k3`列，并根据查询结果创建新表`order_new`，然后将查询结果插入新表。另外，将新表的列名指定为`a`、`b`和`c`。

```SQL
CREATE TABLE order_new (a, b, c)
AS SELECT k1, k2, k3 FROM order;
```

或者

```SQL
CREATE TABLE order_new
AS SELECT k1 AS a, k2 AS b, k3 AS c FROM order;
```

示例3：同步查询表`employee`中`salary`列的最大值，并根据查询结果创建新表`employee_new`，然后将查询结果插入新表。另外，将新表的列名指定为`salary_max`。

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

示例4：使用CTAS创建Primary Key表。请注意，Primary Key表中的数据行数可能少于查询结果。这是因为[Primary Key](../../../table_design/table_types/primary_key_table.md)表只存储具有相同主键的一组行中最新的数据行。

```SQL
CREATE TABLE employee_new
PRIMARY KEY(order_id)
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

示例5：同步查询包括`lineorder`、`customer`、`supplier`和`part`四个表，并根据查询结果创建新表`lineorder_flat`，然后将查询结果插入新表。另外，为新表指定分区方法和分桶方法。

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
    p.P_CONTAINER FROM lineorder AS l
INNER JOIN customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
INNER JOIN supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
INNER JOIN part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```

```SQL

异步查询表`order_detail`，并根据查询结果创建新表`order_statistics`，然后将查询结果插入新表。


SUBMIT TASK AS CREATE TABLE order_statistics AS SELECT COUNT(*) as count FROM order_detail;

+-------------------------------------------+-----------+
| TaskName                                  | Status    |
+-------------------------------------------+-----------+
| ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2 | SUBMITTED |

+-------------------------------------------+-----------+

```


查询任务信息。

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

查询任务运行状态。

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;

-- 任务运行状态

QUERY_ID: 37bd2b63-eba8-11ec-8d41-bab8ee315bf2
TASK_NAME: ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2

CREATE_TIME: 2022-06-14 14:07:06

FINISH_TIME: 2022-06-14 14:07:07
STATE: SUCCESS
DATABASE: 