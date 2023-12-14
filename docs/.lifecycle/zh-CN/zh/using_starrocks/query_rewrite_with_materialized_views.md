---
displayed_sidebar: "Chinese"
---

# 物化视图查询改写

This article describes how to use StarRocks' asynchronous materialized views to rewrite and accelerate queries.

## 概述

StarRocks's asynchronous materialized views use mainstream SPJG (select-project-join-group-by) pattern transparent query rewriting algorithm. Without modifying the query statement, StarRocks can automatically rewrite the query on the base table to the query on the materialized view. With the pre-computed results included, materialized views can help you significantly reduce computation costs and greatly accelerate query execution.

The query rewrite function based on asynchronous materialized views is particularly useful in the following scenarios:

- **Metric Pre-aggregation**

  If you need to process high-dimensional data, you can use materialized views to create a pre-aggregated metric layer.

- **Wide Table Join**

  Materialized views allow you to transparently accelerate queries containing large wide table joins in complex scenarios.

- **Data Lake Acceleration**

  Building materialized views based on External Catalog can easily accelerate queries for data in the data lake.

> **Note**
>
> Asynchronous materialized views built on JDBC Catalog tables do not currently support query rewriting.

### 功能特点

StarRocks' automatic query rewriting function for asynchronous materialized views has the following features:

- **Strong Data Consistency**: If the base table is a StarRocks internal table, StarRocks can ensure that the results obtained from query rewriting through materialized views are consistent with directly querying the base table.
- **Staleness Rewrite**: StarRocks supports staleness rewrite, allowing a certain degree of tolerance for data expiration to cope with frequent data changes.
- **Multi-table Join**: StarRocks' asynchronous materialized views support various types of joins, including some complex join scenarios such as View Delta Join and Join Derivation Rewrite, which can be used to accelerate queries involving large wide tables.
- **Aggregation Rewrite**: StarRocks can rewrite queries with aggregation operations to improve report performance.
- **Nested Materialized Views**: StarRocks supports complex query rewriting based on nested materialized views, extending the range of queries that can be rewritten.
- **Union Rewrite**: You can combine the Union rewrite feature with the materialized view partition time-to-live (TTL) to separate hot and cold data, allowing you to query hot data from the materialized view and historical data from the base table.
- **Materialized View Construction Based on Views**: You can accelerate queries in scenarios based on view modeling.
- **Materialized View Construction Based on External Catalog**: You can accelerate queries in the data lake using this feature.
- **Complex Expression Rewrite**: Support calling functions and arithmetic operations in expressions to meet complex analysis and calculation requirements.

These features will be detailed in the following sections.

## Join 改写

StarRocks supports rewriting queries with various types of joins, including Inner Join, Cross Join, Left Outer Join, Full Outer Join, Right Outer Join, Semi Join, and Anti Join. The following example shows the rewriting of a join query. Created the following base tables:

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

Based on the base tables above, create the following materialized view:

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY HASH(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, lo_partkey, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

This materialized view can rewrite the following query:

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey;
```

The original query plan and the rewritten plan are as follows:

![Rewrite-1](../assets/Rewrite-1.png)

StarRocks supports rewriting join queries with complex expressions, such as arithmetic operations, string functions, date functions, CASE WHEN expressions, and predicates OR. For example, the materialized view mentioned above can rewrite the following query:

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

In addition to the regular scenarios, StarRocks also supports rewriting join queries in more complex scenarios.

### Query Delta Join Rewrite

Query Delta Join refers to the situation where the tables joined in the query are a superset of the tables joined in the materialized view. For example, the following query joins the tables `lineorder`, `customer`, and `part`. If the materialized view `join_mv1` only contains the join of `lineorder` and `customer`, StarRocks can use `join_mv1` to rewrite the query.

Example:

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address, p_name
FROM
    lineorder INNER JOIN customer ON lo_custkey = c_custkey
    INNER JOIN part ON lo_partkey = p_partkey;
```

The original query plan and the rewritten plan are as follows:

![Rewrite-2](../assets/Rewrite-2.png)

### View Delta Join Rewrite

View Delta Join refers to the situation where the tables joined in the query are a subset of the tables joined in the materialized view. This feature is typically used in scenarios involving large wide tables. For example, in the context of the Star Schema Benchmark (SSB), you can improve query performance by creating materialized views that join all tables. Testing found that after transparently rewriting queries through materialized views, the query performance of multi-table joins can reach the same performance level as querying the corresponding large wide table.

To enable View Delta Join rewrite, it is necessary to ensure that the materialized view contains a 1:1 Cardinality Preservation Join that is not present in the query. The following nine types of joins that satisfy the following constraint conditions are considered to be Cardinality Preservation Join and can be used to enable View Delta Join rewriting:

![Rewrite-3](../assets/Rewrite-3.png)

以 SSB 测试为例，创建以下基表：

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
"unique_constraints" = "c_custkey"   -- 指定唯一键。
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
"unique_constraints" = "d_datekey"   -- 指定唯一键。
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
"unique_constraints" = "s_suppkey"   -- 指定唯一键。
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
"unique_constraints" = "p_partkey"   -- 指定唯一键。
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
PARTITION p7 VALUES [("1998-01-01"), ("1999-01-01")))
DISTRIBUTED BY HASH(lo_orderkey) BUCKETS 48
PROPERTIES (
"foreign_key_constraints" = "
    (lo_custkey) REFERENCES customer(c_custkey);
    (lo_partkey) REFERENCES part(p_partkey);
    (lo_suppkey) REFERENCES supplier(s_suppkey)" -- 指定外键约束。
);
```

创建 Join 表 `lineorder`、表 `customer`、表 `supplier`、表 `part` 和表 `dates` 的物化视图 `lineorder_flat_mv`：

```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY LO_ORDERDATE
REFRESH MANUAL
PROPERTIES (
    "partition_refresh_number"="1"
)
AS SELECT /*+ SET_VAR(query_timeout = 7200) */     -- 设置刷新超时时间。
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
```sql
       d.D_HOLIDAYFL         AS D_HOLIDAYFL,
       d.D_WEEKDAYFL         AS D_WEEKDAYFL
   FROM lineorder            AS l
       INNER JOIN customer   AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
       INNER JOIN supplier   AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
       INNER JOIN part       AS p ON p.P_PARTKEY = l.LO_PARTKEY
       INNER JOIN dates      AS d ON l.LO_ORDERDATE = d.D_DATEKEY;    
```

SSB Q2.1 涉及四个表的 Join，但与物化视图 `lineorder_flat_mv` 相比，缺少了 `customer` 表。在 `lineorder_flat_mv` 中，`lineorder INNER JOIN customer` 本质上是一个 Cardinality Preservation Join。因此逻辑上，可以消除该 Join 而不影响查询结果。因此，Q2.1 可以使用 `lineorder_flat_mv` 进行改写。

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

其原始查询计划和改写后的计划如下：

![Rewrite-4](../assets/Rewrite-4.png)

同样，SSB 中的其他查询也可以通过使用 `lineorder_flat_mv` 进行透明改写，从而优化查询性能。

### Join 派生改写

Join 派生是指物化视图和查询中的 Join 类型不一致，但物化视图的 Join 结果包含查询 Join 结果的情况。目前支持两种情景：三表或以上 Join 和两表 Join 的情景。

- **情景一：三表或以上 Join**

  假设物化视图包含表 `t1` 和表 `t2` 之间的 Left Outer Join，以及表 `t2` 和表 `t3` 之间的 Inner Join。两个 Join 的条件都包括来自表 `t2` 的列。

  而查询则包含 `t1` 和 `t2` 之间的 Inner Join，以及 `t2` 和 `t3` 之间的 Inner Join。两个 Join 的条件都包括来自表 `t2` 的列。

  在这种情况下，上述查询可以通过物化视图改写。这是因为在物化视图中，首先执行 Left Outer Join，然后执行 Inner Join。Left Outer Join 生成的右表没有匹配结果（即右表中的列为 NULL）。这些结果在执行 Inner Join 期间被过滤掉。因此，物化视图和查询的逻辑是等效的，可以对查询进行改写。

  示例：

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

  `join_mv5` 可改写以下查询：

  ```SQL
  SELECT lo_orderkey, lo_orderdate, lo_linenumber, lo_revenue, c_custkey, c_address, p_name
  FROM customer INNER JOIN lineorder
  ON c_custkey = lo_custkey
  INNER JOIN part
  ON p_partkey = lo_partkey;
  ```

  其原始查询计划和改写后的计划如下：

  ![Rewrite-5](../assets/Rewrite-5.png)

  同样，如果物化视图定义为 `t1 INNER JOIN t2 INNER JOIN t3`，而查询为 `LEFT OUTER JOIN t2 INNER JOIN t3`，那么查询也可以被改写。而且，在涉及超过三个表的情况下，也具备上述的改写能力。

- **情景二：两表 Join**

  两表 Join 的派生改写支持以下几种细分场景：

  ![Rewrite-6](../assets/Rewrite-6.png)

  在场景一至九中，需要向改写结果补偿过滤谓词，以确保语义等效性。例如，创建以下物化视图：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv3
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  则 `join_mv3` 可以改写以下查询，其查询结果需补偿谓词 `c_custkey IS NOT NULL`：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  其原始查询计划和改写后的计划如下：

  ![Rewrite-7](../assets/Rewrite-7.png)

  在场景十中， 需要 Left Outer Join 查询中包含右表中 `IS NOT NULL` 的过滤谓词，如 `=`、`<>`、`>`、`<`、`<=`、`>=`、`LIKE`、`IN`、`NOT LIKE` 或 `NOT IN`。例如，创建以下物化视图：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv4
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  则 `join_mv4` 可以改写以下查询，其中 `customer.c_address = "Sb4gxKs7"` 为 `IS NOT NULL` 谓词：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey
  WHERE customer.c_address = "Sb4gxKs7";
  ```

  其原始查询计划和改写后的计划如下：

  ![Rewrite-8](../assets/Rewrite-8.png)

## 聚合改写

StarRocks 异步物化视图的多表聚合查询改写支持所有聚合函数，包括 bitmap_union、hll_union 和 percentile_union 等。例如，创建以下物化视图：

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

该物化视图可以改写以下查询：

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

其原始查询计划和改写后的计划如下：

![Rewrite-9](../assets/Rewrite-9.png)

以下各节详细阐述了聚合改写功能可用的场景。

### 聚合上卷改写
StarRocks支持通过聚合上卷改写查询，即StarRocks可以使用通过`GROUP BY a,b`子句创建的异步物化视图改写带有`GROUP BY a`子句的聚合查询。例如，`agg_mv1`可以改写以下查询：

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

其原始查询计划和改写后的计划如下：

![Rewrite-10](../assets/Rewrite-10.png)

> **说明**
>
> 当前暂不支持 grouping set、grouping set with rollup 以及 grouping set with cube 的改写。

仅有部分聚合函数支持聚合上卷查询改写。下表展示了原始查询中的聚合函数与用于构建物化视图的聚合函数之间的对应关系。您可以根据自己的业务场景，选择相应的聚合函数构建物化视图。

| **原始查询聚合函数**              | **支持Aggregate Rollup的物化视图构建聚合函数**    |
| ---------------------------- | ------------------------------------------ |
| sum                          | sum                                        |
| count                        | count                                      |
| min                          | min                                        |
| max                          | max                                        |
| avg                          | sum / count                                |
| bitmap_union,bitmap_union_count,count(distinct) | bitmap_union                               |
| hll_raw_agg,hll_union_agg,ndv,approx_count_distinct | hll_union                                  |
| percentile_approx,percentile_union  | percentile_union                           |

没有相应GROUP BY列的DISTINCT聚合无法使用聚合上卷查询改写。但是，从StarRocks v3.1开始，如果聚合上卷对应DISTINCT聚合函数的查询没有GROUP BY列，但有等价的谓词，该查询也可以被相关物化视图重写，因为StarRocks可以将等价谓词转换为GROUP BY常量表达式。

在以下示例中，StarRocks可以使用物化视图`order_agg_mv1`改写对应查询Query：

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

-- Query
SELECT
    order_date,
    count(distinct client_id) 
FROM order_list WHERE order_date='2023-07-03';
```

### COUNT DISTINCT改写

StarRocks支持将COUNT DISTINCT计算改写为BITMAP类型的计算，从而使用物化视图实现高性能、精确的去重。例如，创建以下物化视图：

```SQL
CREATE MATERIALIZED VIEW distinct_mv
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, bitmap_union(to_bitmap(lo_custkey)) AS distinct_customer
FROM lineorder
GROUP BY lo_orderkey;
```

该物化视图可以改写以下查询：

```SQL
SELECT lo_orderkey, count(distinct lo_custkey) 
FROM lineorder 
GROUP BY lo_orderkey;
```

## 嵌套物化视图改写

StarRocks支持使用嵌套物化视图改写查询。例如，创建以下物化视图`join_mv2`、`agg_mv2`和`agg_mv3`：

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

其关系如下：

![Rewrite-11](../assets/Rewrite-11.png)

`agg_mv3`可改写以下查询：

```SQL
SELECT 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder INNER JOIN customer
ON lo_custkey = c_custkey
GROUP BY lo_orderkey;
```

其原始查询计划和改写后的计划如下：

![Rewrite-12](../assets/Rewrite-12.png)

## Union改写

### 谓词Union改写

当物化视图的谓词范围是查询的谓词范围的子集时，可以使用UNION操作改写查询。

例如，创建以下物化视图：

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

该物化视图可以改写以下查询：

```SQL
select 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

其原始查询计划和改写后的计划如下：

![Rewrite-13](../assets/Rewrite-13.png)

其中，`agg_mv5`包含`lo_orderkey < 300000000`的数据，`lo_orderkey >= 300000000`的数据通过直接查询表`lineorder`得到，最终通过Union操作之后再聚合，获取最终结果。

### 分区Union改写

假设基于分区表创建了一个分区物化视图。当查询扫描的分区范围是物化视图最新分区范围的超集时，查询可被UNION改写。

例如，有如下的物化视图`agg_mv4`。基表`lineorder`当前包含分区`p1`至`p7`，物化视图也包含分区`p1`至`p7`。

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

如果`lineorder`新增分区`p8`，其范围为[("19990101"), ("20000101"))，则以下查询可被UNION改写：

```SQL
SELECT 
  lo_orderdate, 
  lo_orderkey, 
  sum(lo_revenue) AS total_revenue, 
  max(lo_discount) AS max_discount 
FROM lineorder
GROUP BY lo_orderkey;
```

其原始查询计划和改写后的计划如下：

![Rewrite-14](../assets/Rewrite-14.png)

如上所示，`agg_mv5`包含来自分区`p1`到`p7`的数据，而分区`p8`的数据来源于`lineorder`。最后，这两组数据使用UNION操作合并。
```SQL
SELECT lo_orderkey, lo_linenumber, lo_custkey, lo_revenue
FROM lineorder;
```

According to the above view, create a materialized view:

```SQL
CREATE MATERIALIZED VIEW join_mv1
DISTRIBUTED BY hash(lo_orderkey)
AS
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name
FROM lineorder_view1 INNER JOIN customer_view1
ON lo_custkey = c_custkey;
```

During the query rewriting process, queries against `customer_view1` and `lineorder_view1` will be automatically expanded to the base tables and then transparently matched for rewriting.

## Building Materialized Views Based on External Catalog

StarRocks supports building asynchronous materialized views on external data sources based on Hive Catalog, Hudi Catalog, and Iceberg Catalog, and supports transparent query rewriting. Materialized views based on External Catalog support most query rewriting functions, but have the following limitations:

- Materialized views created based on Hudi, Iceberg, and JDBC Catalogs do not support Union rewriting.
- Materialized views created based on Hudi, Iceberg, and JDBC Catalogs do not support View Delta Join rewriting.
- Materialized views created based on Hudi, Iceberg, and JDBC Catalogs do not support partition incremental refresh.

## Setting Materialized View Query Rewriting

You can set asynchronous materialized view query rewriting through the following Session variables.

| **Variable**                                    | **Default Value** | **Description**                                                     |
| ------------------------------------------- | ---------- | ------------------------------------------------------------ |
| enable_materialized_view_union_rewrite | true | Whether to enable materialized view Union rewriting.  |
| enable_rule_based_materialized_view_rewrite | true | Whether to enable rule-based materialized view query rewriting, mainly used for handling single-table query rewriting. |
| nested_mv_rewrite_max_level | 3 | The maximum level of nested materialized views available for query rewriting. Type: INT. Value range: [1, +∞). A value of `1` means that only materialized views created based on base tables can be used for query rewriting. |

## Verify Whether Query Rewriting Is Effective

You can use the EXPLAIN statement to view the corresponding Query Plan. If the `TABLE` under the `OlapScanNode` item is the name of the corresponding asynchronous materialized view, it means that the query has been rewritten based on the asynchronous materialized view.

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

## Disable Query Rewriting

StarRocks enables query rewriting for asynchronous materialized views created based on the Default Catalog by default. You can disable this feature by setting the Session variable `enable_materialized_view_rewrite` to `false`.

For asynchronous materialized views created based on external catalogs, you can disable this feature by setting the materialized view Property `force_external_table_query_rewrite` to `false` through [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md).

## Limitations

In terms of materialized view query rewriting capability, StarRocks currently has the following limitations:

- StarRocks does not support the rewriting of non-deterministic functions, including rand, random, uuid, and sleep.
- StarRocks does not support the rewriting of window functions.
- If the materialized view definition statement contains LIMIT, ORDER BY, UNION, EXCEPT, INTERSECT, MINUS, GROUPING SETS, WITH CUBE, or WITH ROLLUP, it cannot be used for rewriting.
- Materialized views based on External Catalogs do not guarantee strong consistency of query results.
- Asynchronous materialized views built on tables based on JDBC Catalogs currently do not support query rewriting.