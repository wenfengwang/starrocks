---
displayed_sidebar: English
---

# 使用物化视图重写查询

本主题描述了如何利用 StarRocks 的异步物化视图来重写和加速查询。

## 概述

StarRocks 的异步物化视图采用了广泛采用的基于 SPJG（select-project-join-group-by）形式的透明查询重写算法。无需修改查询语句，StarRocks 可以自动将对基表的查询改写为对应的包含预计算结果的物化视图的查询。因此，物化视图可以帮助您显著降低计算成本，并大大加快查询执行速度。

基于异步物化视图的查询重写功能在以下场景中特别有用：

- **度量的预聚合**

  如果您处理的是高维数据，则可以使用物化视图创建预聚合度量层。

- **宽表的联接**

  物化视图允许您在复杂场景中透明地加速对多个大型宽表的查询。

- **数据湖中的查询加速**

  构建基于外部目录的物化视图可以轻松加速对数据湖中数据的查询。

  > **注意**
  >
  > 在 JDBC 目录中的基表上创建的异步物化视图不支持查询重写。

### 特性

StarRocks 基于物化视图的异步查询重写功能具有以下特性：

- **数据一致性强**：如果基表是原生表，StarRocks 会确保通过物化视图查询重写得到的结果与直接查询基表返回的结果一致。
- **过期重写**：StarRocks 支持过期重写，允许您容忍一定级别的数据过期，以应对频繁更改数据的情况。
- **多表联接**：StarRocks 的异步物化视图支持多种类型的联接，包括一些复杂的联接场景，如视图增量联接和可派生联接，从而加速涉及大型宽表的查询。
- **聚合重写**：StarRocks 可以重写带有聚合的查询，以提升报表性能。
- **嵌套物化视图**：StarRocks 支持基于嵌套物化视图重写复杂查询，扩大了可重写的查询范围。
- **并集重写**：您可以将并集重写功能与物化视图分区的TTL（Time-to-Live）结合使用，实现热数据和冷数据的分离，从而可以从物化视图查询热数据，从基表查询历史数据。
- **视图上的物化视图**：您可以加速基于视图数据建模的查询。
- **外部目录的物化视图**：您可以加速数据湖中的查询。
- **复杂表达式重写**：它可以处理复杂的表达式，包括函数调用和算术运算，满足高级分析和计算要求。

这些特性将在以下各节中详细阐述。

## 联接重写

StarRocks 支持使用多种类型的联接进行重写查询，包括内连接、交叉连接、左外连接、全外连接、右外连接、半连接和反连接。

以下是使用联接重写查询的示例。创建两个基表，如下所示：

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

使用上述基表，可以按如下方式创建物化视图：

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

![重写-1](../assets/Rewrite-1.png)

StarRocks 支持使用复杂的表达式重写联接查询，例如算术运算、字符串函数、日期函数、CASE WHEN 表达式和 OR 谓词。例如，上面的物化视图可以重写以下查询：

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

除了常规场景外，StarRocks 还支持在更复杂的场景下重写联接查询。

### 查询增量联接重写

查询增量联接是指查询中联接的表是物化视图中联接的表的超集。例如，请考虑以下涉及三个表的联接的查询：`lineorder`、 `customer`和 `part`。如果物化视图 `join_mv1` 仅包含 `lineorder` 和 `customer` 的联接，StarRocks 可以使用 `join_mv1` 重写查询。

例：

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address, p_name
FROM
    lineorder INNER JOIN customer ON lo_custkey = c_custkey
    INNER JOIN part ON lo_partkey = p_partkey;
```

其原始查询计划和重写后的查询计划如下：

![重写-2](../assets/Rewrite-2.png)

### 视图增量联接重写

视图增量联接是指查询中联接的表是物化视图中联接的表的子集。此功能通常用于涉及大型宽表的场景。例如，在星型架构基准（SSB）的上下文中，您可以创建一个联接所有表的物化视图，以提高查询性能。通过测试发现，通过物化视图透明重写查询后，多表联接的查询性能可以达到与查询对应大宽表相同的性能水平。

要执行视图增量联接重写，物化视图必须包含查询中不存在的 1:1 基数保留联接。以下是被视为基数保留联接的九种联接类型，满足其中任何一种类型都会启用视图增量联接重写：

![重写-3](../assets/Rewrite-3.png)

以SSB测试为例，创建如下基表：

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
  lo_orderdate       DATE          NOT NULL, -- 指定为 NOT NULL。
  lo_orderkey        INT(11)       NOT NULL,
  lo_linenumber      TINYINT       NOT NULL,
  lo_custkey         INT(11)       NOT NULL, -- 指定为 NOT NULL。
  lo_partkey         INT(11)       NOT NULL, -- 指定为 NOT NULL。
  lo_suppkey         INT(11)       NOT NULL, -- 指定为 NOT NULL。
  lo_orderpriority   VARCHAR(100)  NOT NULL,
  lo_shippriority    TINYINT       NOT NULL,
  lo_quantity        TINYINT       NOT NULL,
  lo_extendedprice   INT(11)       NOT NULL,
  lo_ordtotalprice   INT(11)       NOT NULL,
  lo_discount        TINYINT       NOT NULL,
  lo_revenue         INT(11)       NOT NULL,
  lo_supplycost      INT(11)       NOT NULL,
  lo_tax             TINYINT       NOT NULL,
  lo_commitdate      DATE          NOT NULL,
  lo_shipmode        VARCHAR(100)  NOT NULL
) ENGINE=OLAP
DUPLICATE KEY(lo_orderdate,lo_orderkey)
PARTITION BY RANGE(lo_orderdate)
(PARTITION p1 VALUES [("0000-01-01"), ("1993-01-01")),
PARTITION p2 VALUES [("1993-01-01"), ("1994-01-01")),
PARTITION p3 VALUES [("1994-01-01"), ("1995-01-01")),
PARTITION p4 VALUES [("1995-01-01"), ("1996-01-01")),
PARTITION p5 VALUES [("1996-01-01"), ("1997-01-01")),
PARTITION p6 VALUES [("1997-01-01"), ("1998-01-01")),
PARTITION p7 VALUES [("1998-01-01"), ("1999-01-01"))
DISTRIBUTED BY HASH(lo_orderkey) BUCKETS 48
PROPERTIES (
"foreign_key_constraints" = "
    (lo_custkey) REFERENCES customer(c_custkey);
    (lo_partkey) REFERENCES part(p_partkey);
    (lo_suppkey) REFERENCES supplier(s_suppkey)" -- 指定外键。
);
```

创建物化视图 `lineorder_flat_mv`，它联接了 `lineorder`、`customer`、`supplier`、`part` 和 `dates`：

```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY LO_ORDERDATE
REFRESH MANUAL
PROPERTIES (
    "partition_refresh_number"="1"
)
AS SELECT /*+ SET_VAR(query_timeout = 7200) */     -- 为刷新操作设置超时时间。
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

SSB Q2.1 涉及连接四个表，但与物化视图相比，缺少了 `customer` 表。在 `lineorder_flat_mv` 中，`lineorder INNER JOIN customer` 本质上是一个基数保留连接。因此，从逻辑上讲，可以在不影响查询结果的情况下消除此连接。因此，Q2.1 可以使用 `lineorder_flat_mv` 进行重写。

SSB 问题 2.1：

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

![重写-4](../assets/Rewrite-4.png)

同样，SSB 中的其他查询也可以透明地使用 `lineorder_flat_mv` 进行重写，从而优化查询性能。

### 联接派生重写

联接派生是指物化视图与查询的联接类型不一致，但物化视图的联接结果包含查询的联接结果的情况。目前，它支持两种方案 - 联接三个或更多表，以及联接两个表。

- **方案一：联接三个或更多表**

  假设物化视图包含表 `t1` 和 `t2` 之间的左外连接，以及表 `t2` 和 `t3` 之间的内连接。在这两个联接中，联接条件都包括表 `t2` 中的列。

  另一方面，查询包含 t1 和 t2 之间的内部联接，以及 t2 和 t3 之间的内部联接。在这两个联接中，联接条件都包括 t2 中的列。

  在这种情况下，可以使用物化视图重写查询。这是因为在物化视图中，首先执行“左外连接”，然后执行“内连接”。由左外联接生成的右表没有匹配结果（即，右表中的列为 NULL）。这些结果随后在内部联接期间被过滤掉。因此，物化视图和查询的逻辑是等价的，可以重写查询。

  例：

  创建物化视图 `join_mv5`：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv5
  PARTITION BY lo_orderdate
  DISTRIBUTED BY hash(lo_orderkey)
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

  ![重写-5](../assets/Rewrite-5.png)

  同样，如果物化视图定义为 `t1 INNER JOIN t2 INNER JOIN t3`，而查询是 `LEFT OUTER JOIN t2 INNER JOIN t3`，查询也可以重写。此外，此重写功能扩展到涉及三个以上表的方案。

- **方案二：联接两个表**

  涉及两个表的联接派生性重写功能支持以下特定情况：

  ![重写-6](../assets/Rewrite-6.png)

  在情况 1 到 9 中，必须将过滤谓词添加到重写的结果中，以确保语义等效。例如，按如下方式创建物化视图：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv3
  DISTRIBUTED BY hash(lo_orderkey)
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

  ![重写-7](../assets/Rewrite-7.png)

  在情况 10 中，左外联接查询必须在右表中包含筛选谓词 `IS NOT NULL`，例如，`=`, `<>`, `>`, `<`, `<=`, `>=`, `LIKE`, `IN`, `NOT LIKE`, `NOT IN`。例如，按如下方式创建物化视图：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv4
  DISTRIBUTED BY hash(lo_orderkey)
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

  ![重写-8](../assets/Rewrite-8.png)

## 聚合重写

StarRocks 的异步物化视图支持使用所有可用的聚合函数（包括 bitmap_union、hll_union 和 percentile_union）重写多表聚合查询。例如，按如下方式创建物化视图：

```SQL
CREATE MATERIALIZED VIEW agg_mv1
DISTRIBUTED BY hash(lo_orderkey)
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

![重写-9](../assets/Rewrite-9.png)

以下各节阐述了聚合重写功能可能有用的方案。

### 聚合汇总重写

StarRocks 支持使用 Aggregation Rollup 重写查询，即 StarRocks 可以使用使用子句 `GROUP BY a` 创建的异步物化视图来重写带有子句 `GROUP BY a,b` 的聚合查询。例如，可以使用以下命令重写以下查询 `agg_mv1`：

```SQL
SELECT 
  lo_orderkey, 
  c_name, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey, c_name;
```

其原始查询计划和重写后的查询计划如下：

![重写-10](../assets/Rewrite-10.png)

> **注意**
>
> 目前，不支持重写分组集、使用汇总的分组集或使用多维数据集的分组集。

只有某些聚合函数支持使用聚合汇总进行查询重写。在前面的示例中，如果物化视图 `order_agg_mv` 使用 `count(distinct client_id)` 代替 `bitmap_union(to_bitmap(client_id))`，StarRocks 无法使用 Aggregate Rollup 重写查询。

下表显示了原始查询中的聚合函数与用于构建物化视图的聚合函数之间的对应关系。您可以根据业务场景选择对应的聚合函数来构建物化视图。

| **原始查询中支持的聚合函数**   | **物化视图中支持的聚合函数** |
| ------------------------------ | ---------------------------- |
| sum                            | sum                          |
| count                          | count                        |
| min                            | min                          |
| max                            | max                          |
| avg                            | sum / count                  |
| bitmap_union, bitmap_union_count, count(distinct) | bitmap_union |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union |
| percentile_approx, percentile_union | percentile_union |

没有相应 GROUP BY 列的 DISTINCT 聚合无法使用聚合汇总重写。但是，从 StarRocks v3.1 开始，如果使用 Aggregate Rollup DISTINCT 聚合函数的查询没有 GROUP BY 列，而是有相等的谓词，也可以通过相关的物化视图进行重写，因为 StarRocks 可以将相等的谓词转换为 GROUP BY 常量表达式。

在以下示例中，StarRocks 可以使用物化视图重写查询 `order_agg_mv1`。

```SQL
CREATE MATERIALIZED VIEW order_agg_mv1
DISTRIBUTED BY HASH(`order_id`) BUCKETS 12
REFRESH ASYNC START('2022-09-01 10:00:00') EVERY (interval 1 day)
AS
SELECT
    order_date,
    count(distinct client_id) 
FROM order_list 
GROUP BY order_date;


-- 查询
SELECT
    order_date,
    count(distinct client_id) 
FROM order_list WHERE order_date='2023-07-03';
```

### COUNT DISTINCT 重写

StarRocks 支持将 COUNT DISTINCT 计算重写为基于位图的计算，从而实现高性能、精确的去重。例如，按如下方式创建物化视图：

```SQL
CREATE MATERIALIZED VIEW distinct_mv
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, bitmap_union(to_bitmap(lo_custkey)) AS distinct_customer
FROM lineorder
GROUP BY lo_orderkey;
```

它可以重写以下查询：

```SQL
SELECT lo_orderkey, count(distinct lo_custkey) 
FROM lineorder 
GROUP BY lo_orderkey;
```

## 嵌套物化视图重写

StarRocks 支持使用嵌套物化视图重写查询。例如，按如下方式创建物化视图 `join_mv2`、 `agg_mv2` 和 `agg_mv3`：

```SQL
CREATE MATERIALIZED VIEW join_mv2
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;


CREATE MATERIALIZED VIEW agg_mv2
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT 
  lo_orderkey, 
  lo_linenumber, 
  c_name, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM join_mv2
GROUP BY lo_orderkey, lo_linenumber, c_name;

CREATE MATERIALIZED VIEW agg_mv3
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT 
  lo_orderkey, 
  sum(total_revenue) AS total_revenue, 
  max(max_discount) AS max_discount 
FROM agg_mv2
GROUP BY lo_orderkey;
```

它们之间的关系如下：

![重写-11](../assets/Rewrite-11.png)

`agg_mv3` 可以重写以下查询：

```SQL
SELECT 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey;
```

其原始查询计划和重写后的查询计划如下：

![重写-12](../assets/Rewrite-12.png)

## 联合重写

### 谓词联合重写

当物化视图的谓词范围是查询谓词范围的子集时，可以使用 UNION 操作重写查询。

例如，按如下方式创建物化视图：

```SQL
CREATE MATERIALIZED VIEW agg_mv4
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
WHERE lo_orderkey < 300000000
GROUP BY lo_orderkey;
```

它可以重写以下查询：

```SQL
select 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

其原始查询计划和重写后的查询计划如下：

![重写-13](../assets/Rewrite-13.png)

在此上下文中，`agg_mv5` 包含数据 `lo_orderkey < 300000000`。数据 `lo_orderkey >= 300000000` 直接来自基表 `lineorder`。最后，这两组数据使用 UNION 操作合并，然后进行聚合以获得最终结果。

### 分区联合重写

假设您基于分区表创建了一个分区物化视图。当可重写查询扫描的分区范围是物化视图的最新分区范围的超集时，查询将使用 UNION 操作重写。

例如，考虑以下物化视图 `agg_mv4`。其基表 `lineorder` 目前包含从 `p1` 到 `p7` 的分区，而物化视图也包含从 `p1` 到 `p7` 的分区。

```SQL
CREATE MATERIALIZED VIEW agg_mv5
DISTRIBUTED BY hash(lo_orderkey)
PARTITION BY RANGE(lo_orderdate)
REFRESH MANUAL
AS
SELECT 
  lo_orderdate, 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

如果向 `lineorder` 添加了一个新分区 `p8`，其分区范围为 `[("19990101"), ("20000101"))`，则以下查询可以使用 UNION 操作重写：

```SQL
SELECT 
  lo_orderdate, 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

其原始查询计划和重写后的查询计划如下：

![重写-14](../assets/Rewrite-14.png)

如上图所示，`agg_mv5` 包含来自 `p1` 到 `p7` 的数据，而来自 `p8` 的数据直接从 `lineorder` 查询。最后，这两组数据使用 UNION 操作合并。

## 基于视图的物化视图重写

StarRocks 支持基于视图创建物化视图。后续针对这些视图的查询可以透明地重写。

例如，创建以下视图：

```SQL
CREATE VIEW customer_view1 
AS
SELECT c_custkey, c_name, c_address
FROM customer;

CREATE VIEW lineorder_view1
AS
SELECT lo_orderkey, lo_linenumber, lo_custkey, lo_revenue
FROM lineorder;
```

然后，基于这些视图创建以下物化视图：

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name
FROM lineorder_view1 INNER JOIN customer_view1
ON lo_custkey = c_custkey;
```

在对 `customer_view1` 和 `lineorder_view1` 的查询重写期间，这些查询会自动扩展到基表，然后透明地匹配和重写。

## 基于外部目录的物化视图重写

StarRocks 支持在 Hive 目录、Hudi 目录和 Iceberg 目录上构建异步物化视图，并透明地使用它们重写查询。基于目录的外部物化视图支持大多数查询重写功能，但存在一些限制：

- Hudi、Iceberg 或基于 JDBC 目录的物化视图不支持 Union 重写。
- Hudi、Iceberg 或基于 JDBC 目录的物化视图不支持 View Delta Join 重写。
- Hudi、Iceberg 或基于 JDBC 目录的物化视图不支持分区的增量刷新。

## 配置查询重写

您可以通过以下会话变量配置异步物化视图查询重写：

| **变量**                                | **默认值** | **描述**                                              |
| ------------------------------------------- | ----------- | ------------------------------------------------------------ |
| enable_materialized_view_union_rewrite      | true        | 用于控制是否启用物化视图 Union 查询重写的布尔值。 |
| enable_rule_based_materialized_view_rewrite | true        | 用于控制是否启用基于规则的物化视图查询重写的布尔值。该变量主要用于单表查询重写。 |
| nested_mv_rewrite_max_level                 | 3           | 可用于查询重写的嵌套物化视图的最大级别。类型：INT。 范围：[1， +∞）。的值`1`指示在其他物化视图上创建的物化视图将不用于查询重写。 |

## 检查查询是否被重写

您可以通过使用 EXPLAIN 语句查看查询计划的查询来检查查询是否已重写。如果 `OlapScanNode` 部分下的 `TABLE` 字段显示对应物化视图的名称，则表示查询已基于物化视图重写。

```Plain
mysql> EXPLAIN SELECT 
    order_id, sum(goods.price) AS total 
    FROM order_list INNER JOIN goods 
    ON goods.item_id1 = order_list.item_id2 
    GROUP BY order_id;
+------------------------------------+
| Explain String                     |
+------------------------------------+
| PLAN FRAGMENT 0                    |
|  OUTPUT EXPRS:1: order_id | 8: sum |
|   PARTITION: RANDOM                |
|                                    |
|   RESULT SINK                      |
|                                    |
|   1:Project                        |
|   |  <slot 1> : 9: order_id        |
|   |  <slot 8> : 10: total          |
|   |                                |
|   0:OlapScanNode                   |
|      TABLE: order_mv               |
|      PREAGGREGATION: ON            |
|      partitions=1/1                |
|      rollup: order_mv              |
|      tabletRatio=0/12              |
|      tabletList=                   |
|      cardinality=3                 |
|      avgRowSize=4.0                |
|      numNodes=0                    |
+------------------------------------+
20 rows in set (0.01 sec)
```

## 禁用查询重写

默认情况下，StarRocks 会为基于默认目录创建的异步物化视图启用查询重写。您可以通过将会话变量设置为 `false` 来禁用此功能 `enable_materialized_view_rewrite`。

对于基于外部目录创建的异步物化视图，可以通过使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 将物化视图属性设置为 `force_external_table_query_rewrite` `false` 来禁用此功能。

## 局限性


就基于物化视图的查询重写而言，StarRocks 目前存在以下限制：

- StarRocks 不支持使用非确定性函数进行查询重写，包括 rand、random、uuid 和 sleep 等函数。
- StarRocks 不支持对包含窗口函数的查询进行重写。
- 包含 LIMIT、ORDER BY、UNION、EXCEPT、INTERSECT、MINUS、GROUPING SETS、WITH CUBE 或 WITH ROLLUP 的语句所定义的物化视图，无法用于查询重写。
- 无法保证基表和基于外部目录构建的物化视图之间查询结果的强一致性。
- 在 JDBC 目录中创建的基表上的异步物化视图，不支持查询重写。