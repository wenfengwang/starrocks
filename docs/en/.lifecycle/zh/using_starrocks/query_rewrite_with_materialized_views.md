---
displayed_sidebar: English
---

# 使用物化视图重写查询

本主题介绍如何利用 StarRocks 的异步物化视图来重写和加速查询。

## 概述

StarRocks 的异步物化视图采用了广泛认可的基于 SPJG（select-project-join-group-by）形式的透明查询重写算法。无需修改查询语句，StarRocks 能够自动将针对基表的查询重写为针对包含预计算结果的相应物化视图的查询。因此，物化视图可以帮助您显著降低计算成本，并显著加快查询执行速度。

基于异步物化视图的查询重写功能在以下场景中特别有用：

- **指标预聚合**

  如果您正在处理高维数据，可以使用物化视图创建预聚合指标层。

- **宽表连接**

  物化视图允许您在复杂场景中透明地加速多个大型宽表的连接查询。

- **数据湖中的查询加速**

  构建基于外部目录的物化视图可以轻松加速对数据湖中数据的查询。

    > **注意**
    > 在 JDBC 目录中的基表上创建的异步物化视图不支持查询重写。

### 特性

StarRocks 基于异步物化视图的自动查询重写具有以下特性：

- **强数据一致性**：如果基表是原生表，StarRocks 确保通过物化视图重写得到的查询结果与直接查询基表返回的结果一致。
- **过时重写**：StarRocks 支持过时重写，允许您容忍一定程度的数据过期，以应对数据频繁变更的场景。
- **多表连接**：StarRocks 的异步物化视图支持多种类型的连接，包括一些复杂的连接场景，如 View Delta Joins 和 Derivable Joins，使您能够在涉及大型宽表的场景中加速查询。
- **聚合重写**：StarRocks 可以重写带有聚合的查询，以提升报告性能。
- **嵌套物化视图**：StarRocks 支持基于嵌套物化视图重写复杂查询，扩大了可重写查询的范围。
- **Union 重写**：您可以将 Union 重写功能与物化视图分区的 TTL（Time-to-Live）结合使用，实现热数据和冷数据的分离，从而从物化视图中查询热数据，从基表中查询历史数据。
- **视图上的物化视图**：您可以加速基于视图的数据建模场景中的查询。
- **外部目录的物化视图**：您可以加速数据湖中的查询。
- **复杂表达式重写**：它可以处理复杂表达式，包括函数调用和算术运算，满足高级分析和计算需求。

这些特性将在以下部分中详细阐述。

## 加入重写

StarRocks 支持使用各种类型的连接重写查询，包括内连接、交叉连接、左外连接、全外连接、右外连接、半连接和反连接。

以下是使用连接重写查询的示例。创建两个基表如下：

```SQL
CREATE TABLE customer (
  c_custkey     INT(11)     NOT NULL,
  c_name        VARCHAR(26) NOT NULL,
  c_address     VARCHAR(41) NOT NULL,
  c_city        VARCHAR(11) NOT NULL,
  c_nation      VARCHAR(16) NOT NULL,
  c_region      VARCHAR(13) NOT NULL,
  c_phone       VARCHAR(16) NOT NULL,
  c_mktsegment  VARCHAR(11) NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(c_custkey)
DISTRIBUTED BY HASH(c_custkey) BUCKETS 12;

CREATE TABLE lineorder (
  lo_orderkey         INT(11) NOT NULL,
  lo_linenumber       INT(11) NOT NULL,
  lo_custkey          INT(11) NOT NULL,
  lo_partkey          INT(11) NOT NULL,
  lo_suppkey          INT(11) NOT NULL,
  lo_orderdate        INT(11) NOT NULL,
  lo_orderpriority    VARCHAR(16) NOT NULL,
  lo_shippriority     INT(11) NOT NULL,
  lo_quantity         INT(11) NOT NULL,
  lo_extendedprice    INT(11) NOT NULL,
  lo_ordtotalprice    INT(11) NOT NULL,
  lo_discount         INT(11) NOT NULL,
  lo_revenue          INT(11) NOT NULL,
  lo_supplycost       INT(11) NOT NULL,
  lo_tax              INT(11) NOT NULL,
  lo_commitdate       INT(11) NOT NULL,
  lo_shipmode         VARCHAR(11) NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(lo_orderkey)
DISTRIBUTED BY HASH(lo_orderkey) BUCKETS 48;
```

有了上述基表，您可以创建物化视图，如下所示：

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY HASH(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, lo_partkey, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

这样的物化视图可以重写以下查询：

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

![Rewrite-1](../assets/Rewrite-1.png)

StarRocks 支持使用复杂表达式重写连接查询，例如算术运算、字符串函数、日期函数、CASE WHEN 表达式和 OR 谓词。例如，上述物化视图可以重写以下查询：

```SQL
SELECT 
    lo_orderkey, 
    lo_linenumber, 
    (2 * lo_revenue + 1) * lo_linenumber, 
    upper(c_name), 
    substr(c_address, 3)
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

除了常规场景，StarRocks 还支持在更复杂的场景中重写连接查询。

### 查询 Delta Join 重写

查询 Delta Join 指的是查询中连接的表是物化视图中连接的表的超集的场景。例如，考虑以下涉及三个表的查询：`lineorder`、`customer` 和 `part`。如果物化视图 `join_mv1` 仅包含 `lineorder` 和 `customer` 的连接，StarRocks 可以使用 `join_mv1` 重写查询。

示例：

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address, p_name
FROM
    lineorder INNER JOIN customer ON lo_custkey = c_custkey
    INNER JOIN part ON lo_partkey = p_partkey;
```

其原始查询计划和重写后的查询计划如下：

![Rewrite-2](../assets/Rewrite-2.png)

### 视图 Delta Join 重写

视图 Delta Join 指的是查询中连接的表是物化视图中连接的表的子集的场景。这个特性通常用于涉及大型宽表的场景。例如，在星型模式基准（SSB）的上下文中，您可以创建一个连接所有表的物化视图以提高查询性能。通过测试发现，通过物化视图透明重写查询后，多表连接的查询性能可以达到与查询相应大型宽表相同的水平。

要执行视图 Delta Join 重写，物化视图必须包含查询中不存在的 1:1 基数保留连接。以下是被视为基数保留连接的九种连接类型，满足其中任何一种都可以启用视图 Delta Join 重写：

![Rewrite-3](../assets/Rewrite-3.png)

以 SSB 测试为例，创建如下基表：

```SQL
CREATE TABLE customer (
  c_custkey         INT(11)       NOT NULL,
  c_name            VARCHAR(26)   NOT NULL,
  c_address         VARCHAR(41)   NOT NULL,
  c_city            VARCHAR(11)   NOT NULL,
  c_nation          VARCHAR(16)   NOT NULL,
  c_region          VARCHAR(13)   NOT NULL,
  c_phone           VARCHAR(16)   NOT NULL,
  c_mktsegment      VARCHAR(11)   NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(c_custkey)
DISTRIBUTED BY HASH(c_custkey) BUCKETS 12
PROPERTIES (
"unique_constraints" = "c_custkey"   -- 指定唯一约束。
);

CREATE TABLE dates (
  d_datekey          DATE          NOT NULL,
  d_date             VARCHAR(20)   NOT NULL,
  d_dayofweek        VARCHAR(10)   NOT NULL,
  d_month            VARCHAR(11)   NOT NULL,
  d_year             INT(11)       NOT NULL,
  d_yearmonthnum     INT(11)       NOT NULL,
  d_yearmonth        VARCHAR(9)    NOT NULL,
  d_daynuminweek     INT(11)       NOT NULL,
  d_daynuminmonth    INT(11)       NOT NULL,
  d_daynuminyear     INT(11)       NOT NULL,
  d_monthnuminyear   INT(11)       NOT NULL,
  d_weeknuminyear    INT(11)       NOT NULL,
  d_sellingseason    VARCHAR(14)   NOT NULL,
  d_lastdayinweekfl  INT(11)       NOT NULL,
  d_lastdayinmonthfl INT(11)       NOT NULL,
  d_holidayfl        INT(11)       NOT NULL,
  d_weekdayfl        INT(11)       NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(d_datekey)
DISTRIBUTED BY HASH(d_datekey) BUCKETS 1
PROPERTIES (
"unique_constraints" = "d_datekey"   -- 指定唯一约束。
);

CREATE TABLE supplier (
  s_suppkey          INT(11)       NOT NULL,
  s_name             VARCHAR(26)   NOT NULL,
  s_address          VARCHAR(26)   NOT NULL,
  s_city             VARCHAR(11)   NOT NULL,
  s_nation           VARCHAR(16)   NOT NULL,
  s_region           VARCHAR(13)   NOT NULL,
  s_phone            VARCHAR(16)   NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(s_suppkey)
DISTRIBUTED BY HASH(s_suppkey) BUCKETS 12
PROPERTIES (
"unique_constraints" = "s_suppkey"   -- 指定唯一约束。
);

CREATE TABLE part (
  p_partkey          INT(11)       NOT NULL,
  p_name             VARCHAR(23)   NOT NULL,
  p_mfgr             VARCHAR(7)    NOT NULL,
  p_category         VARCHAR(8)    NOT NULL,
  p_brand            VARCHAR(10)   NOT NULL,
  p_color            VARCHAR(12)   NOT NULL,
  p_type             VARCHAR(26)   NOT NULL,
  p_size             TINYINT(11)   NOT NULL,
  p_container        VARCHAR(11)   NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(p_partkey)
DISTRIBUTED BY HASH(p_partkey) BUCKETS 12
PROPERTIES (
"unique_constraints" = "p_partkey"   -- 指定唯一约束。
);

CREATE TABLE lineorder (
  lo_orderdate       DATE          NOT NULL, -- 标记为 NOT NULL。
  lo_orderkey        INT(11)       NOT NULL,
  lo_linenumber      TINYINT       NOT NULL,
  lo_custkey         INT(11)       NOT NULL, -- 标记为 NOT NULL。
  lo_partkey         INT(11)       NOT NULL, -- 标记为 NOT NULL。
  lo_suppkey         INT(11)       NOT NULL, -- 标记为 NOT NULL。
  lo_orderpriority   VARCHAR(100)  NOT NULL,
  lo_shippriority    TINYINT       NOT NULL,
  lo_quantity        TINYINT       NOT NULL,
  lo_extendedprice   INT(11)       NOT NULL,
  lo_ordtotalprice   INT(11)       NOT NULL,
  lo_discount        TINYINT       NOT NULL,
  lo_revenue         INT(11)       NOT NULL,
  lo_supplycost      INT(11)       NOT NULL,
```
```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY RANGE(LO_ORDERDATE)
REFRESH MANUAL
PROPERTIES (
    "partition_refresh_number"="1"
)
AS SELECT /*+ SET_VAR(query_timeout = 7200) */     -- 设置刷新操作的超时时间。
       l.LO_ORDERDATE        AS LO_ORDERDATE,
       l.LO_ORDERKEY         AS LO_ORDERKEY,
       l.LO_LINENUMBER       AS LO_LINENUMBER,
       l.LO_CUSTKEY          AS LO_CUSTKEY,
       l.LO_PARTKEY          AS LO_PARTKEY,
       l.LO_SUPPKEY          AS LO_SUPPKEY,
       l.LO_ORDERPRIORITY    AS LO_ORDERPRIORITY,
       l.LO_SHIPPRIORITY     AS LO_SHIPPRIORITY,
       l.LO_QUANTITY         AS LO_QUANTITY,
       l.LO_EXTENDEDPRICE    AS LO_EXTENDEDPRICE,
       l.LO_ORDTOTALPRICE    AS LO_ORDTOTALPRICE,
       l.LO_DISCOUNT         AS LO_DISCOUNT,
       l.LO_REVENUE          AS LO_REVENUE,
       l.LO_SUPPLYCOST       AS LO_SUPPLYCOST,
       l.LO_TAX              AS LO_TAX,
       l.LO_COMMITDATE       AS LO_COMMITDATE,
       l.LO_SHIPMODE         AS LO_SHIPMODE,
       c.C_NAME              AS C_NAME,
       c.C_ADDRESS           AS C_ADDRESS,
       c.C_CITY              AS C_CITY,
       c.C_NATION            AS C_NATION,
       c.C_REGION            AS C_REGION,
       c.C_PHONE             AS C_PHONE,
       c.C_MKTSEGMENT        AS C_MKTSEGMENT,
       s.S_NAME              AS S_NAME,
       s.S_ADDRESS           AS S_ADDRESS,
       s.S_CITY              AS S_CITY,
       s.S_NATION            AS S_NATION,
       s.S_REGION            AS S_REGION,
       s.S_PHONE             AS S_PHONE,
       p.P_NAME              AS P_NAME,
       p.P_MFGR              AS P_MFGR,
       p.P_CATEGORY          AS P_CATEGORY,
       p.P_BRAND             AS P_BRAND,
       p.P_COLOR             AS P_COLOR,
       p.P_TYPE              AS P_TYPE,
       p.P_SIZE              AS P_SIZE,
       p.P_CONTAINER         AS P_CONTAINER,
       d.D_DATE              AS D_DATE,
       d.D_DAYOFWEEK         AS D_DAYOFWEEK,
       d.D_MONTH             AS D_MONTH,
       d.D_YEAR              AS D_YEAR,
       d.D_YEARMONTHNUM      AS D_YEARMONTHNUM,
       d.D_YEARMONTH         AS D_YEARMONTH,
       d.D_DAYNUMINWEEK      AS D_DAYNUMINWEEK,
       d.D_DAYNUMINMONTH     AS D_DAYNUMINMONTH,
       d.D_DAYNUMINYEAR      AS D_DAYNUMINYEAR,
       d.D_MONTHNUMINYEAR    AS D_MONTHNUMINYEAR,
       d.D_WEEKNUMINYEAR     AS D_WEEKNUMINYEAR,
       d.D_SELLINGSEASON     AS D_SELLINGSEASON,
       d.D_LASTDAYINWEEKFL   AS D_LASTDAYINWEEKFL,
       d.D_LASTDAYINMONTHFL  AS D_LASTDAYINMONTHFL,
       d.D_HOLIDAYFL         AS D_HOLIDAYFL,
       d.D_WEEKDAYFL         AS D_WEEKDAYFL
   FROM lineorder            AS l
       INNER JOIN customer   AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
       INNER JOIN supplier   AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
       INNER JOIN part       AS p ON p.P_PARTKEY = l.LO_PARTKEY
       INNER JOIN dates      AS d ON l.LO_ORDERDATE = d.D_DATEKEY;    
```

SSB Q2.1 涉及连接四个表，但与物化视图 `lineorder_flat_mv` 相比，它缺少 `customer` 表。在 `lineorder_flat_mv` 中，`lineorder INNER JOIN customer` 本质上是一个基数保留连接。因此，从逻辑上来说，这个连接是可以消除的，不会影响查询结果。因此，可以使用 `lineorder_flat_mv` 重写 Q2.1。

SSB Q2.1：

```SQL
SELECT sum(lo_revenue) AS lo_revenue, d_year, p_brand
FROM lineorder
JOIN dates ON lo_orderdate = d_datekey
JOIN part ON lo_partkey = p_partkey
JOIN supplier ON lo_suppkey = s_suppkey
WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA'
GROUP BY d_year, p_brand
ORDER BY d_year, p_brand;
```

其原始查询计划和重写后的查询计划如下：

![Rewrite-4](../assets/Rewrite-4.png)

同样，SSB中的其他查询也可以使用 `lineorder_flat_mv` 透明地重写，从而优化查询性能。

### Join Derivability 重写

Join Derivability 是指物化视图和查询中的连接类型不一致，但物化视图的连接结果包含查询的连接结果的场景。目前支持两种场景：连接三个或更多表、连接两个表。

- **场景一：连接三个或更多表**

  假设物化视图包含表 `t1` 和 `t2` 之间的 Left Outer Join 以及表 `t2` 和 `t3` 之间的 Inner Join。在这两个连接中，连接条件包括 `t2` 中的列。

  另一方面，查询包含 `t1` 和 `t2` 之间的 Inner Join，以及 `t2` 和 `t3` 之间的 Inner Join。在这两个连接中，连接条件包括 `t2` 中的列。

  在这种情况下，可以使用物化视图重写查询。这是因为在物化视图中，先执行 Left Outer Join，然后执行 Inner Join。Left Outer Join 生成的右表没有匹配结果（即右表列为 NULL）。这些结果随后在内连接期间被过滤掉。因此，物化视图和查询的逻辑是等价的，查询可以重写。

  示例：

  创建物化视图 `join_mv5`：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv5
  PARTITION BY RANGE(lo_orderdate)
  DISTRIBUTED BY HASH(lo_orderkey)
  PROPERTIES (
    "partition_refresh_number" = "1"
  )
  AS
  SELECT lo_orderkey, lo_orderdate, lo_linenumber, lo_revenue, c_custkey, c_address, p_name
  FROM customer LEFT OUTER JOIN lineorder
  ON c_custkey = lo_custkey
  INNER JOIN part
  ON p_partkey = lo_partkey;
  ```

  `join_mv5` 可以重写以下查询：

  ```SQL
  SELECT lo_orderkey, lo_orderdate, lo_linenumber, lo_revenue, c_custkey, c_address, p_name
  FROM customer INNER JOIN lineorder
  ON c_custkey = lo_custkey
  INNER JOIN part
  ON p_partkey = lo_partkey;
  ```

  其原始查询计划和重写后的查询计划如下：

  ![Rewrite-5](../assets/Rewrite-5.png)

  同样，如果物化视图定义为 `t1 INNER JOIN t2 INNER JOIN t3`，并且查询为 `t1 LEFT OUTER JOIN t2 INNER JOIN t3`，查询也可以重写。此外，这种重写能力还扩展到涉及三个以上表的场景。

- **场景二：连接两个表**

  涉及两个表的 Join Derivability Rewrite 功能支持以下特定情况：

  ![Rewrite-6](../assets/Rewrite-6.png)

  对于情况 1 到 9，必须在重写的结果中添加过滤谓词以确保语义等价。例如，创建一个物化视图如下：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv3
  DISTRIBUTED BY HASH(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  可以使用 `join_mv3` 重写以下查询，并将谓词 `c_custkey IS NOT NULL` 添加到重写的结果中：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  其原始查询计划和重写后的查询计划如下：

  ![Rewrite-7](../assets/Rewrite-7.png)

  在情况 10 中，左外连接查询必须在右表中包含过滤谓词 `IS NOT NULL`，例如 `=`、`<>`、`>`、`<`、`<=`、`>=`、`LIKE`、`IN`、`NOT LIKE` 或 `NOT IN`。例如，创建一个物化视图如下：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv4
  DISTRIBUTED BY HASH(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  `join_mv4` 可以重写以下查询，其中 `customer.c_address = "Sb4gxKs7"` 是过滤谓词 `IS NOT NULL`：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey
  WHERE customer.c_address = "Sb4gxKs7";
  ```

  其原始查询计划和重写后的查询计划如下：

  ![Rewrite-8](../assets/Rewrite-8.png)

## 聚合重写

StarRocks 的异步物化视图支持使用所有可用的聚合函数（包括 `bitmap_union`、`hll_union` 和 `percentile_union`）重写多表聚合查询。例如，创建一个物化视图如下：

```SQL
CREATE MATERIALIZED VIEW agg_mv1
DISTRIBUTED BY HASH(lo_orderkey)
AS
SELECT 
  lo_orderkey, 
  lo_linenumber, 
  c_name, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey, lo_linenumber, c_name;
```

它可以重写以下查询：

```SQL
SELECT 
  lo_orderkey, 
  lo_linenumber, 
  c_name, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey, lo_linenumber, c_name;
```

其原始查询计划和重写后的查询计划如下：

![Rewrite-9](../assets/Rewrite-9.png)

以下部分阐述了聚合重写功能可能有用的场景。

### 聚合 Rollup 重写

StarRocks 支持使用 Aggregation Rollup 重写查询，即 StarRocks 可以使用通过 `GROUP BY a,b` 子句创建的异步物化视图重写具有 `GROUP BY a` 子句的聚合查询。例如，可以使用 `agg_mv1` 重写以下查询：

```SQL
SELECT 
```

在基于物化视图的查询重写方面，StarRocks 当前存在以下限制：

- StarRocks 不支持重写包含非确定性函数的查询，这些函数包括 `rand`、`random`、`uuid` 和 `sleep`。
- StarRocks 不支持重写包含窗口函数的查询。
- 定义包含 `LIMIT`、`ORDER BY`、`UNION`、`EXCEPT`、`INTERSECT`、`MINUS`、`GROUPING SETS`、`WITH CUBE` 或 `WITH ROLLUP` 语句的物化视图不能用于查询重写。
- 在基表和基于外部目录构建的物化视图之间，不保证查询结果的强一致性。