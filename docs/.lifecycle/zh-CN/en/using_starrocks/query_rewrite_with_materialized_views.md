---
displayed_sidebar: "Chinese"
---

# 使用物化视图重写查询

本主题介绍了如何利用 StarRocks 的异步物化视图来重写和加速查询。

## 概述

StarRocks 的异步物化视图采用了基于 SPJG（select-project-join-group-by）形式的广泛采用的透明查询重写算法。在无需修改查询语句的情况下，StarRocks 可以自动将针对基本表的查询重写为针对包含预先计算结果的相应物化视图的查询。因此，物化视图可以帮助您显著减少计算成本，并大大加速查询执行。

基于异步物化视图的查询重写功能在以下场景中特别有用：

- **度量的预聚合**

  如果您处理的是高维度数据，可以使用物化视图创建一个预聚合的度量层。

- **宽表的连接**

  物化视图允许您在复杂场景中透明加速对多个大宽表进行连接的查询。

- **数据湖中的查询加速**

  构建基于外部目录的物化视图可以轻松加速对数据湖中数据的查询。

  > **注意**
  >
  > 在 JDBC 目录中创建的基本表上的异步物化视图不支持查询重写。

### 特性

StarRocks 基于异步物化视图的自动查询重写特性具有以下特点：

- **强一致性数据**：如果基本表为原生表，StarRocks 确保通过基于物化视图的查询重写获得的结果与直接针对基本表的查询返回的结果保持一致。
- **陈旧重写**：StarRocks 支持陈旧重写，允许您容忍一定水平的数据过期以应对频繁数据更改的情况。
- **多表连接**：StarRocks 的异步物化视图支持各种类型的连接，包括一些复杂的连接场景，比如视图增量连接和可推导连接，允许您加速涉及大宽表的查询场景。
- **聚合重写**：StarRocks 可以重写带有聚合的查询以提高报表性能。
- **嵌套物化视图**：StarRocks 支持基于嵌套物化视图重写复杂查询，扩大可重写查询范围。
- **Union 重写**：您可以将 Union 重写功能与物化视图分区的 TTL（生存时间）结合使用，实现热数据和冷数据分离，允许您从物化视图查询热数据和从基本表查询历史数据。
- **视图上的物化视图**：您可以加速基于视图的数据建模场景中的查询。
- **外部目录中的物化视图**：您可以加速数据湖中的查询。
- **复杂表达式重写**：它可以处理复杂的表达式，包括函数调用和算术操作，满足高级分析和计算需求。

这些特性将在以下章节详细介绍。

## 连接重写

StarRocks 支持重写带有各种类型连接的查询，包括内连接、交叉连接、左外连接、全外连接、右外连接、半连接和反连接。

以下是重写带有连接查询的示例。创建两个基本表如下：

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

通过上述基本表，您可以创建以下物化视图：

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

StarRocks 支持重写连接查询中的复杂表达式，比如算术运算、字符串函数、日期函数、CASE WHEN 表达式和 OR 谓词。例如，上述物化视图可以重写以下查询：

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

除了常规场景外，StarRocks 还支持在更复杂场景中重写连接查询。

### 查询增量连接重写

查询增量连接指的是查询中连接的表是物化视图中连接的表的超集的场景。例如，考虑以下涉及三个表（`lineorder`、`customer` 和 `part`）连接的查询。如果物化视图 `join_mv1` 仅包含 `lineorder` 和 `customer` 的连接，StarRocks 可以通过 `join_mv1` 重写查询。

示例：

```SQL
SELECT lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address, p_name
FROM
    lineorder INNER JOIN customer ON lo_custkey = c_custkey
    INNER JOIN part ON lo_partkey = p_partkey;
```

原始查询计划和重写后的查询计划如下：

![重写-2](../assets/Rewrite-2.png)

### 视图增量连接重写

视图增量连接指的是查询中连接的表是物化视图中连接的表的子集的场景。这个功能通常用于涉及大宽表的场景。例如，在星型模式基准（SSB）的上下文中，您可以创建一个将所有表连接起来以提高查询性能的物化视图。通过测试，已发现在透明重写查询后，多表连接的查询性能可以达到与查询相应大宽表的性能同等水平。

要执行视图增量连接重写，物化视图必须包含在查询中不存在的保持 1:1 基数的连接。以下是被认为是基数保持连接的九种类型的连接，满足其中任何一种即可启用视图增量连接重写：

![重写-3](../assets/Rewrite-3.png)

以 SSB 测试为例，创建以下基本表：

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
"unique_constraints" = "c_custkey"   -- Specify the unique constraints.
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
```SQL
创建名为lineorder_flat_mv的物化视图
按哈希方式分布(LO_ORDERDATE，LO_ORDERKEY) 桶48
按LO_ORDERDATE分区
手动刷新
属性 (
    "partition_refresh_number"="1"
)
作为 选择 /*+ SET_VAR(query_timeout = 7200) */     -- 为刷新操作设置超时。
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
   从 lineorder            AS l
       内连接 客户      AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
       内连接 供应商    AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
       内连接 零件      AS p ON p.P_PARTKEY = l.LO_PARTKEY
       内连接 日期      AS d ON l.LO_ORDERDATE = d.D_DATEKEY;    
```

SSB Q2.1涉及连接四个表，但与物化视图`lineorder_flat_mv`相比，缺少`customer`表。在`lineorder_flat_mv`中，`lineorder INNER JOIN customer`本质上是一个基数保持连接。因此，在不影响查询结果的情况下，可以消除此连接。因此，可以使用`lineorder_flat_mv`重写Q2.1。

SSB Q2.1:

```SQL
选择 sum(lo_revenue) AS lo_revenue, d_year, p_brand
从 lineorder
内连接 日期 ON lo_orderdate = d_datekey
内连接 零件 ON lo_partkey = p_partkey
内连接 供应商 ON lo_suppkey = s_suppkey
其中 p_category = 'MFGR#12' 和 s_region = 'AMERICA'
按 d_year, p_brand 分组
按 d_year, p_brand 排序;
```

其原始查询计划和重写后的查询计划如下:

![Rewrite-4](../assets/Rewrite-4.png)
```
类似地，SSB中的其他查询也可以通过使用“lineorder_flat_mv”透明地重写，从而优化查询性能。

### 联接可导重写

联接可导是指物化视图中的联接类型与查询中的不一致，但物化视图的联接结果包含查询联接的结果的情况。目前，它支持两种情形 - 连接三个或更多表，以及连接两个表。

- **情形一：连接三个或更多表**

  假设物化视图包含表“t1”和“t2”的左外连接，以及表“t2”和“t3”的内连接。在这两个连接中，连接条件均包括来自“t2”的列。

  另一方面，查询包含表t1和t2之间的内连接，以及表t2和t3之间的内连接。在这两个连接中，连接条件均包括来自t2的列。

  在这种情况下，查询可以使用物化视图进行重写。这是因为在物化视图中，左外连接先执行，然后是内连接。左外连接生成的右表对于匹配（即，右表中的列为NULL）没有结果。在内连接过程中随后过滤掉这些结果。因此，物化视图的逻辑与查询是等价的，可以进行重写。

  例如：

  创建物化视图“join_mv5”：

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

  “join_mv5”可以重写以下查询：

  ```SQL
  SELECT lo_orderkey, lo_orderdate, lo_linenumber, lo_revenue, c_custkey, c_address, p_name
  FROM customer INNER JOIN lineorder
  ON c_custkey = lo_custkey
  INNER JOIN part
  ON p_partkey = lo_partkey;
  ```

  其原始查询计划和重写后的查询计划如下：

  ![Rewrite-5](../assets/Rewrite-5.png)

  类似地，如果物化视图被定义为“t1 INNER JOIN t2 INNER JOIN t3”，而查询是“LEFT OUTER JOIN t2 INNER JOIN t3”，查询也可以进行重写。此外，这种重写能力扩展到涉及三个以上表的情形。

- **情形二：连接两个表**

  与两个表有关的联接可导重写功能支持以下具体情形：

  ![Rewrite-6](../assets/Rewrite-6.png)

  在情形1至9中，必须向重写结果中添加过滤谓词，以确保语义等价。例如，如下创建物化视图：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv3
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  以下查询可以使用“join_mv3”进行重写，并向重写结果添加谓词“c_custkey IS NOT NULL”：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  其原始查询计划和重写后的查询计划如下：

  ![Rewrite-7](../assets/Rewrite-7.png)

  在情形10中，左外连接查询必须在右表中包含过滤谓词“IS NOT NULL”，例如`=`, `<>`, `>`, `<`, `<=`, `>=`, `LIKE`, `IN`, `NOT LIKE`或`NOT IN`。例如，如下创建物化视图：

  ```SQL
  CREATE MATERIALIZED VIEW join_mv4
  DISTRIBUTED BY hash(lo_orderkey)
  AS
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder INNER JOIN customer
  ON lo_custkey = c_custkey;
  ```

  “join_mv4”可以重写以下查询，其中“customer.c_address = "Sb4gxKs7"`是过滤谓词“IS NOT NULL”：

  ```SQL
  SELECT lo_orderkey, lo_linenumber, lo_revenue, c_custkey, c_address
  FROM lineorder LEFT OUTER JOIN customer
  ON lo_custkey = c_custkey
  WHERE customer.c_address = "Sb4gxKs7";
  ```

  其原始查询计划和重写后的查询计划如下：

  ![Rewrite-8](../assets/Rewrite-8.png)

## 聚合重写

StarRocks的异步物化视图支持使用所有可用的聚合函数重写多表聚合查询，包括bitmap_union、hll_union和percentile_union。例如，如下创建物化视图：

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

![Rewrite-9](../assets/Rewrite-9.png)

下面的部分详细说明了聚合重写功能可以提供帮助的情形。

### 聚合Rollup重写

StarRocks支持重写具有聚合Rollup的查询，也就是说，StarRocks可以通过使用具有`GROUP BY a,b`子句创建的异步物化视图来重写具有`GROUP BY a`子句的聚合查询。例如，以下查询可以使用“agg_mv1”进行重写：

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

![Rewrite-10](../assets/Rewrite-10.png)

> **注意**
>
> 目前，不支持重写分组集、带Rollup的分组集或带立体的分组集。

只有特定的聚合函数支持使用聚合Rollup进行查询重写。在上述示例中，如果物化视图“order_agg_mv”使用`count(distinct client_id)`而不是`bitmap_union(to_bitmap(client_id))`，StarRocks无法重写具有聚合Rollup的查询。

下表显示了原始查询中的聚合函数与用于构建物化视图的聚合函数之间的对应关系。根据业务情况，可以选择相应的聚合函数来构建一个物化视图。

| **原始查询中支持的聚合函数** | **物化视图中支持的聚合Rollup函数** |
| --------------------------------- | -------------------------------------------- |
| sum                               | sum                                          |
| count                             | count                                        |
| min                               | min                                          |
| max                               | max                                          |
| avg                               | sum / count                                  |
| bitmap_union、bitmap_union_count、count(distinct) | bitmap_union   |
| hll_raw_agg、hll_union_agg、ndv、approx_count_distinct | hll_union  |
| percentile_approx、percentile_union                | percentile_union  |

不带相应GROUP BY列的DISTINCT聚合无法使用聚合Rollup进行重写。但是，从StarRocks v3.1开始，如果具有聚合Rollup DISTINCT聚合函数的查询没有GROUP BY列但存在相等谓词，它也可以通过相关物化视图进行重写，因为StarRocks可以将相等谓词转换为GROUP BY常量表达式。

在以下示例中，StarRocks可以使用物化视图“order_agg_mv1”重写查询。

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
选择
    order_date，
    计数(不同的 client_id) 
从 订单列表
其中 订单日期='2023-07-03';

### COUNT DISTINCT 重写

StarRocks 支持将 COUNT DISTINCT 计算重写为基于位图的计算，使用物化视图实现高性能、精确去重。例如，创建一个如下的物化视图：

```SQL
创建物化视图 distinct_mv
根据 散列(lo_orderkey) 分布
如
选择 lo_orderkey, bitmap_union(to_bitmap(lo_custkey)) 作为 distinct_customer
从 lineorder
分组 按 lo_orderkey;

它可以重写以下查询：

```SQL
选择 lo_orderkey, 计数(不同的 lo_custkey) 
从 lineorder 
按 lo_orderkey 分组;
```

## 嵌套物化视图重写

StarRocks 支持使用嵌套物化视图重写查询。例如，创建物化视图 `join_mv2`，`agg_mv2`，和 `agg_mv3` 如下：

```SQL
创建物化视图 join_mv2
根据 散列(lo_orderkey) 分布
如
选择 lo_orderkey, lo_linenumber, lo_revenue, c_name, c_address
从 lineorder 内连接 customer
在 lo_custkey = c_custkey;


创建物化视图 agg_mv2
根据 散列(lo_orderkey) 分布
如
选择 
  lo_orderkey, 
  lo_linenumber, 
  c_name, 
  sum(lo_revenue) 作为 total_revenue, 
  max(lo_discount) 作为 max_discount 
从 join_mv2
分组 按 lo_orderkey, lo_linenumber, c_name;

创建物化视图 agg_mv3
根据 散列(lo_orderkey) 分布
如
选择 
  lo_orderkey, 
  sum(total_revenue) 作为 total_revenue, 
  max(max_discount) 作为 max_discount 
从 agg_mv2
分组 按 lo_orderkey;
```

它们的关系如下图所示：

![重写-11](../assets/Rewrite-11.png)

`agg_mv3` 可以重写以下查询：

```SQL
选择 
  lo_orderkey, 
  sum(lo_revenue) 作为 total_revenue, 
  max(lo_discount) 作为 max_discount 
从 lineorder 内连接 customer
在 lo_custkey = c_custkey
按 lo_orderkey 分组;
```

其原始查询计划和重写后的查询计划如下：

![重写-12](../assets/Rewrite-12.png)

## Union 重写

### 谓词 Union 重写

当物化视图的谓词范围是查询的谓词范围的子集时，可以使用 UNION 操作重写查询。

例如，创建一个如下的物化视图：

```SQL
创建物化视图 agg_mv4
根据 散列(lo_orderkey) 分布
如
选择 
  lo_orderkey, 
  sum(lo_revenue) 作为 total_revenue, 
  max(lo_discount) 作为 max_discount 
从 lineorder
在 lo_orderkey < 300000000
分组 按 lo_orderkey;
```

它可以重写以下查询：

```SQL
选择 
  lo_orderkey, 
  sum(lo_revenue) 作为 total_revenue, 
  max(lo_discount) 作为 max_discount 
从 lineorder
按 lo_orderkey 分组;
```

其原始的查询计划和重写后的查询计划如下：

![重写-13](../assets/Rewrite-13.png)

在这种情况下，`agg_mv5` 包含了 `lo_orderkey < 300000000` 的数据。`lo_orderkey >= 300000000` 的数据直接从基表 `lineorder` 获取。最后，这两组数据使用 UNION 操作合并，然后聚合以获得最终结果。

### 分区 Union 重写

假设基于分区表创建了一个分区物化视图。当可重写查询扫描的分区范围是物化视图的最新分区范围的超集时，将使用 UNION 操作重写查询。

例如，考虑以下物化视图 `agg_mv4`。它的基表 `lineorder` 目前包含 `p1` 到 `p7` 的分区，物化视图也包含 `p1` 到 `p7` 的分区。

```SQL
创建物化视图 agg_mv5
根据 散列(lo_orderkey) 分布
按 RANGE(lo_orderdate) 分区
手动刷新
如
选择 
  lo_orderdate, 
  lo_orderkey, 
  sum(lo_revenue) 作为 total_revenue, 
  max(lo_discount) 作为 max_discount 
从 lineorder
分组 按 lo_orderkey;
```

如果新增了一个分区 `p8`，其分区范围为 `[("19990101"), ("20000101"))`，到 `lineorder`，则以下查询可以使用 UNION 操作重写：

```SQL
选择 
  lo_orderdate, 
  lo_orderkey, 
  sum(lo_revenue) 作为 total_revenue, 
  max(lo_discount) 作为 max_discount 
从 lineorder
按 lo_orderkey 分组;
```

其原始的查询计划和重写后的查询计划如下：

![重写-14](../assets/Rewrite-14.png)

如上所示，`agg_mv5` 包含了从 `p1` 到 `p7` 的数据，`p8` 的数据直接从 `lineorder` 查询。最后，这两组数据使用 UNION 操作合并。

## 基于视图的物化视图重写

StarRocks 支持基于视图创建物化视图。随后对视图的查询可以被透明重写。

例如，创建如下视图：

```SQL
创建视图 customer_view1 
如
选择 c_custkey, c_name, c_address
从 customer;

创建视图 lineorder_view1
如
选择 lo_orderkey, lo_linenumber, lo_custkey, lo_revenue
从 lineorder;
```

然后，基于这些视图创建以下物化视图：

```SQL
创建物化视图 join_mv1
根据 散列(lo_orderkey) 分布
如
选择 lo_orderkey, lo_linenumber, lo_revenue, c_name
从 lineorder_view1 内连接 customer_view1
在 lo_custkey = c_custkey;
```

在查询重写过程中，针对 `customer_view1` 和 `lineorder_view1` 的查询会自动扩展到基表，然后透明匹配和重写。

## 基于外部目录的物化视图重写

StarRocks 支持在 Hive 目录、Hudi 目录和 Iceberg 目录上构建异步物化视图，并对其进行透明的查询重写。基于外部目录的物化视图支持大部分查询重写功能，但存在一些限制：

- Hudi、Iceberg 或 JDBC 目录的物化视图不支持 Union 重写。
- Hudi、Iceberg 或 JDBC 目录的物化视图不支持视图 Delta 连接重写。
- Hudi、Iceberg 或 JDBC 目录的物化视图不支持分区的增量刷新。

## 配置查询重写

可以通过以下会话变量配置异步物化视图查询重写：

| **变量**                                      | **默认值** | **描述**                                       |
| ------------------------------------------- | ----------- |------------------------------------------- |
| enable_materialized_view_union_rewrite      | true        | 布尔值，控制是否启用物化视图 Union 查询重写。 |
| enable_rule_based_materialized_view_rewrite | true        | 布尔值，控制是否启用基于规则的物化视图查询重写。该变量主要用于单表查询重写。 |
| nested_mv_rewrite_max_level                 | 3           | 可以用于查询重写的最大嵌套物化视图级别。类型：INT。范围：[1，+∞)。值为 `1` 表示不会使用针对其他物化视图创建的物化视图进行查询重写。 |

## 检查查询是否被重写

可以通过使用 EXPLAIN 语句查看查询计划来检查查询是否被重写。如果 `OlapScanNode` 部分下的 `TABLE` 字段显示了相应物化视图的名称，表示查询已基于物化视图进行了重写。
20 rows in set (0.01 sec)
```

## 禁用查询重写

默认情况下，StarRocks 对基于默认目录创建的异步物化视图启用查询重写。 您可以通过将会话变量 `enable_materialized_view_rewrite` 设置为 `false` 来禁用此功能。

对于基于外部目录创建的异步物化视图，您可以通过使用 [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) 来将物化视图属性 `force_external_table_query_rewrite` 设置为 `false` 以禁用此功能。

## 限制

在基于物化视图的查询重写方面，StarRocks 目前存在以下限制：

- StarRocks 不支持使用非确定性函数（包括 rand、random、uuid 和 sleep）重写查询。
- StarRocks 不支持使用窗口函数重写查询。
- 用包含 LIMIT、ORDER BY、UNION、EXCEPT、INTERSECT、MINUS、GROUPING SETS、WITH CUBE 或 WITH ROLLUP 语句定义的物化视图无法用于查询重写。
- 不能保证在基表和在外部目录上构建的物化视图之间的查询结果的强一致性。
- 在 JDBC 目录上创建的异步基表上不能支持查询重写。