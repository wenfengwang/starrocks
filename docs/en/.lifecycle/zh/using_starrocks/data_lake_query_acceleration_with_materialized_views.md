---
displayed_sidebar: English
---

# 使用物化视图加速数据湖查询

本主题介绍如何使用 StarRocks 的异步物化视图优化数据湖中的查询性能。

StarRocks 提供即开即用的数据湖查询能力，这对于探索性查询和分析湖中数据非常有效。在大多数场景下，[Data Cache](../data_source/data_cache.md) 能够提供块级文件缓存，避免了远程存储抖动和大量 I/O 操作导致的性能下降。

然而，当涉及到使用湖中数据构建复杂且高效的报告或进一步加速这些查询时，您可能仍会面临性能挑战。通过异步物化视图，您可以实现湖上报表和数据应用的更高并发和更佳的查询性能。

## 概述

StarRocks 支持基于外部目录如 Hive 目录、Iceberg 目录和 Hudi 目录构建异步物化视图。基于外部目录的物化视图在以下场景中特别有用：

- **数据湖报告的透明加速**

  为确保数据湖报告的查询性能，数据工程师通常需要与数据分析师紧密合作，深入探讨报告加速层的构建逻辑。如果加速层需要进一步更新，他们必须更新构建逻辑、处理计划和查询语句。

  通过物化视图的查询改写能力，报告加速可以对用户透明且不易察觉。当识别到慢查询时，数据工程师可以分析慢查询的模式并根据需求创建物化视图。应用端的查询随后会被物化视图智能改写和透明加速，允许业务应用或查询语句的逻辑无需修改即可迅速提升查询性能。

- **实时数据与历史数据关联的增量计算**

  假设您的业务应用需要将 StarRocks 原生表中的实时数据与数据湖中的历史数据关联进行增量计算。在这种情况下，物化视图可以提供一个简便的解决方案。例如，如果实时事实表是 StarRocks 中的原生表，而维度表存储在数据湖中，您可以通过构建物化视图将原生表与外部数据源中的表关联，从而轻松进行增量计算。

- **快速构建度量层**

  在处理高维度数据时，计算和处理指标可能会遇到挑战。您可以利用物化视图进行数据预聚合和滚动，以创建一个相对轻量级的度量层。此外，物化视图能够自动刷新，进一步降低度量计算的复杂性。

物化视图、Data Cache 和 StarRocks 中的原生表都是显著提升查询性能的有效方法。下表比较了它们的主要区别：

<table class="comparison">
  <thead>
    <tr>
      <th>&nbsp;</th>
      <th>Data Cache</th>
      <th>Materialized view</th>
      <th>Native table</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>数据加载和更新</b></td>
      <td>查询自动触发数据缓存。</td>
      <td>自动触发刷新任务。</td>
      <td>支持多种导入方法，但需要手动维护导入任务</td>
    </tr>
    <tr>
      <td><b>数据缓存粒度</b></td>
      <td><ul><li>支持块级数据缓存</li><li>遵循 LRU 缓存淘汰机制</li><li>不缓存计算结果</li></ul></td>
      <td>存储预计算的查询结果</td>
      <td>基于表结构存储数据</td>
    </tr>
    <tr>
      <td><b>查询性能</b></td>
      <td colspan="3" style={{textAlign: 'center'}} >Data Cache ≤ Materialized view = Native table</td>
    </tr>
    <tr>
      <td><b>查询语句</b></td>
      <td><ul><li>无需修改针对数据湖的查询语句</li><li>一旦查询命中缓存，即进行计算。</li></ul></td>
      <td><ul><li>无需修改针对数据湖的查询语句</li><li>利用查询改写重用预计算结果</li></ul></td>
      <td>需要修改查询语句以查询原生表</td>
    </tr>
  </tbody>
</table>


<br />


与直接查询数据湖或将数据加载到原生表相比，物化视图提供了几个独特优势：

- **本地存储加速**：物化视图可以利用 StarRocks 的本地存储加速优势，如索引、分区、分桶和并置组，从而比直接从数据湖查询数据获得更好的查询性能。
- **零维护加载任务**：物化视图通过自动刷新任务透明更新数据。无需维护加载任务以执行计划数据更新。此外，基于 Hive 目录的物化视图可以检测数据变更并在分区级别执行增量刷新。
- **智能查询改写**：查询可以透明地改写为使用物化视图。您可以立即从加速中受益，无需修改应用程序使用的查询语句。

<br />


因此，我们建议在以下场景中使用物化视图：

- 即使启用了 Data Cache，查询性能仍无法满足您对查询延迟和并发性的要求。
- 查询涉及可重用组件，如固定的聚合函数或连接模式。
- 数据按分区组织，而查询涉及较高级别的聚合（例如，按天聚合）。

<br />


在以下场景中，我们建议优先通过 Data Cache 进行加速：

- 查询没有许多可重用组件，可能会扫描数据湖中的任何数据。
- 远程存储有显著波动或不稳定，可能会影响访问。

## 创建基于外部目录的物化视图

在外部目录中的表上创建物化视图与在 StarRocks 的原生表上创建类似。您只需根据所使用的数据源设置合适的刷新策略，并手动启用基于外部目录的物化视图的查询改写。

### 选择合适的刷新策略

目前，StarRocks 无法检测 Hudi 目录和 JDBC 目录中的分区级数据变更。因此，一旦触发任务，就会执行全量刷新。

对于 Hive Catalog 和 Iceberg Catalog（从 v3.1.4 版本开始），StarRocks 支持检测分区级别的数据变化。因此，StarRocks 可以：

- 仅刷新数据发生变化的分区，避免全量刷新，减少刷新带来的资源消耗。

- 在查询改写时一定程度上确保数据一致性。如果数据湖中的基表发生数据变化，则查询不会改写为使用物化视图。

    > **注意**
    > 您仍可以选择通过设置属性 `mv_rewrite_staleness_second` 来容忍一定程度的数据不一致性。更多信息，请参见 [CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)。

请注意，如果需要按分区刷新，物化视图的分区键必须包含在基表的分区键中。

对于 Hive 目录，您可以启用 Hive 元数据缓存刷新功能，以便 StarRocks 检测分区级别的数据变化。启用此功能后，StarRocks 定期访问 Hive Metastore Service (HMS) 或 AWS Glue，检查最近查询的热数据的元数据信息。

要启用 Hive 元数据缓存刷新功能，您可以使用 [ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 设置以下 FE 动态配置项：

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| enable_background_refresh_connector_metadata | 在 v3.0 中为 true，在 v2.5 中为 false | 是否启用定期 Hive 元数据缓存刷新。启用后，StarRocks 轮询 Hive 集群的元存储（Hive Metastore 或 AWS Glue），刷新频繁访问的 Hive 目录的缓存元数据以感知数据变化。true 表示启用，false 表示禁用。 |
| background_refresh_metadata_interval_millis | 600000（10 分钟） | 两次连续 Hive 元数据缓存刷新之间的间隔时间。单位：毫秒。 |
| background_refresh_metadata_time_secs_since_last_access_secs | 86400（24 小时） | Hive 元数据缓存刷新任务的过期时间。对于已访问的 Hive 目录，如果超过指定时间没有被访问，StarRocks 将停止刷新其缓存元数据。对于未访问的 Hive 目录，StarRocks 不会刷新其缓存元数据。单位：秒。 |

从 v3.1.4 版本开始，StarRocks 支持在分区级别检测 Iceberg Catalog 的数据变化。目前仅支持 Iceberg V1 表。

### 为基于外部目录的物化视图启用查询改写

默认情况下，StarRocks 不支持对基于 Hudi、Iceberg 和 JDBC 目录构建的物化视图进行查询改写，因为这种场景下的查询改写无法确保结果的强一致性。您可以在创建物化视图时，通过将属性 `force_external_table_query_rewrite` 设置为 `true` 来启用此功能。对于基于 Hive 目录中的表构建的物化视图，默认启用查询改写。

示例：

```SQL
CREATE MATERIALIZED VIEW ex_mv_par_tbl
PARTITION BY emp_date
DISTRIBUTED BY hash(empid)
PROPERTIES (
"force_external_table_query_rewrite" = "true"
) 
AS
select empid, deptno, emp_date
from `hive_catalog`.`emp_db`.`emps_par_tbl`
where empid < 5;
```
```
在涉及查询重写的场景中，如果您使用非常复杂的查询语句构建物化视图，我们建议您将查询语句拆分，并以嵌套的方式构建多个简单的物化视图。嵌套物化视图更加通用，可以适应更广泛的查询模式。

## 最佳实践

在实际业务场景中，您可以通过分析审计日志或[大查询日志](../administration/monitor_manage_big_queries.md#analyze-big-query-logs)来识别具有高执行延迟和资源消耗的查询。您可以进一步使用[查询概要](../administration/query_profile.md)来查明查询速度缓慢的特定阶段。以下部分提供有关如何使用物化视图提高数据湖查询性能的说明和示例。

### 案例一：加速数据湖中的联接计算

您可以使用物化视图来加速数据湖中的联接查询。

假设 Hive 目录上的以下查询特别慢：

```SQL
--Q1
SELECT SUM(lo_extendedprice * lo_discount) AS REVENUE
FROM hive.ssb_1g_csv.lineorder, hive.ssb_1g_csv.dates
WHERE
    lo_orderdate = d_datekey
    AND d_year = 1993
    AND lo_discount BETWEEN 1 AND 3
    AND lo_quantity < 25;

--Q2
SELECT SUM(lo_extendedprice * lo_discount) AS REVENUE
FROM hive.ssb_1g_csv.lineorder, hive.ssb_1g_csv.dates
WHERE
    lo_orderdate = d_datekey
    AND d_yearmonth = 'Jan1994'
    AND lo_discount BETWEEN 4 AND 6
    AND lo_quantity BETWEEN 26 AND 35;

--Q3 
SELECT SUM(lo_revenue), d_year, p_brand
FROM hive.ssb_1g_csv.lineorder, hive.ssb_1g_csv.dates, hive.ssb_1g_csv.part, hive.ssb_1g_csv.supplier
WHERE
    lo_orderdate = d_datekey
    AND lo_partkey = p_partkey
    AND lo_suppkey = s_suppkey
    AND p_brand BETWEEN 'MFGR#2221' AND 'MFGR#2228'
    AND s_region = 'ASIA'
GROUP BY d_year, p_brand
ORDER BY d_year, p_brand;
```

通过分析它们的查询概要，您可能会注意到查询执行时间主要花费在表 `lineorder` 和其他维度表之间的 `lo_orderdate` 列上的哈希联接上。

这里，Q1 和 Q2 在联接 `lineorder` 和 `dates` 后执行聚合，而 Q3 在联接 `lineorder`、`dates`、`part` 和 `supplier` 后执行聚合。

因此，您可以利用 StarRocks 的 [View Delta Join 重写](./query_rewrite_with_materialized_views.md#view-delta-join-rewrite) 功能来构建一个联接 `lineorder`、`dates`、`part` 和 `supplier` 的物化视图。

```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES ( 
    -- Specify the unique constraints.
    "unique_constraints" = "
    hive.ssb_1g_csv.supplier.s_suppkey;
    hive.ssb_1g_csv.part.p_partkey;
    hive.ssb_1g_csv.dates.d_datekey",
    -- Specify the Foreign Keys.
    "foreign_key_constraints" = "
    hive.ssb_1g_csv.lineorder(lo_partkey) REFERENCES hive.ssb_1g_csv.part(p_partkey);
    hive.ssb_1g_csv.lineorder(lo_suppkey) REFERENCES hive.ssb_1g_csv.supplier(s_suppkey);
    hive.ssb_1g_csv.lineorder(lo_orderdate) REFERENCES hive.ssb_1g_csv.dates(d_datekey)",
    -- Enable query rewrite for the external catalog-based materialized view.
    "force_external_table_query_rewrite" = "TRUE"
)
AS SELECT
       l.LO_ORDERDATE AS LO_ORDERDATE,
       l.LO_ORDERKEY AS LO_ORDERKEY,
       l.LO_PARTKEY AS LO_PARTKEY,
       l.LO_SUPPKEY AS LO_SUPPKEY,
       l.LO_QUANTITY AS LO_QUANTITY,
       l.LO_EXTENDEDPRICE AS LO_EXTENDEDPRICE,
       l.LO_DISCOUNT AS LO_DISCOUNT,
       l.LO_REVENUE AS LO_REVENUE,
       s.S_REGION AS S_REGION,
       p.P_BRAND AS P_BRAND,
       d.D_YEAR AS D_YEAR,
       d.D_YEARMONTH AS D_YEARMONTH
   FROM hive.ssb_1g_csv.lineorder AS l
            INNER JOIN hive.ssb_1g_csv.supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
            INNER JOIN hive.ssb_1g_csv.part AS p ON p.P_PARTKEY = l.LO_PARTKEY
            INNER JOIN hive.ssb_1g_csv.dates AS d ON l.LO_ORDERDATE = d.D_DATEKEY;
```

### 案例二：加速数据湖中的聚合和联接聚合

物化视图可用于加速聚合查询，无论它们是在单个表上还是涉及多个表。

- 单表聚合查询

  对于单个表上的典型查询，其查询概要将显示 AGGREGATE 节点消耗大量时间。您可以使用常见的聚合运算符来构造物化视图。

  假设以下是一个慢查询：

  ```SQL
  --Q4
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate
  ORDER BY lo_orderdate limit 100;
  ```

  Q4 计算每日唯一订单数。由于 count distinct 计算的计算成本可能很高，因此您可以创建以下两种类型的物化视图来加速：

  ```SQL
  CREATE MATERIALIZED VIEW mv_2_1 
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  AS 
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  
  CREATE MATERIALIZED VIEW mv_2_2 
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  AS 
  SELECT
  -- lo_orderkey must be the BIGINT type so that it can be used for query rewrite.
  lo_orderdate, bitmap_union(to_bitmap(lo_orderkey))
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  ```

  请注意，在这种情况下，不要创建带有 LIMIT 和 ORDER BY 子句的物化视图，以避免重写失败。有关查询重写限制的更多信息，请参阅 [物化视图查询重写的限制](./query_rewrite_with_materialized_views.md#limitations)。

- 多表聚合查询

  在涉及联接结果聚合的场景中，您可以在现有物化视图上创建嵌套物化视图，这些视图联接表以进一步聚合联接结果。例如，基于案例一的示例，您可以创建以下物化视图来加速 Q1 和 Q2，因为它们的聚合模式相似：

  ```SQL
  CREATE MATERIALIZED VIEW mv_2_3
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  AS 
  SELECT
  lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth, SUM(lo_extendedprice * lo_discount) AS REVENUE
  FROM lineorder_flat_mv
  GROUP BY lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth;
  ```

  当然，也可以在单个物化视图中执行联接和聚合计算。虽然这些类型的物化视图可能有较少的查询重写机会（由于其特定的计算），但它们在聚合后通常占用较少的存储空间。您的选择可以基于您的具体用例。

  ```SQL
  CREATE MATERIALIZED VIEW mv_2_4
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  PROPERTIES (
      "force_external_table_query_rewrite" = "TRUE"
  )
  AS
  SELECT lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth, SUM(lo_extendedprice * lo_discount) AS REVENUE
  FROM hive.ssb_1g_csv.lineorder, hive.ssb_1g_csv.dates
  WHERE lo_orderdate = d_datekey
  GROUP BY lo_orderdate, lo_discount, lo_quantity, d_year, d_yearmonth;
  ```

### 案例三：加速数据湖中的聚合联接

在某些场景下，可能需要先对一张表进行聚合计算，然后再与其他表进行联接查询。为了充分利用 StarRocks 的查询重写功能，我们建议构建嵌套的物化视图。例如：

```SQL
--Q5
SELECT * FROM  (
    SELECT 
      l.lo_orderkey, l.lo_orderdate, c.c_custkey, c_region, sum(l.lo_revenue)
    FROM 
      hive.ssb_1g_csv.lineorder l 
      INNER JOIN (
        SELECT distinct c_custkey, c_region 
        from 
          hive.ssb_1g_csv.customer 
        WHERE 
          c_region IN ('ASIA', 'AMERICA') 
      ) c ON l.lo_custkey = c.c_custkey
      GROUP BY  l.lo_orderkey, l.lo_orderdate, c.c_custkey, c_region
  ) c1 
WHERE 
  lo_orderdate = '19970503';
```

Q5 首先对 `customer` 表进行聚合查询，然后与 `lineorder` 表进行联接和聚合。类似的查询可能涉及 `c_region` 和 `lo_orderdate` 上的不同过滤器。要利用查询重写功能，您可以创建两个物化视图，一个用于聚合，另一个用于联接。

```SQL
--mv_3_1
CREATE MATERIALIZED VIEW mv_3_1
DISTRIBUTED BY HASH(c_custkey)
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES (
    "force_external_table_query_rewrite" = "TRUE"
)
AS
SELECT distinct c_custkey, c_region from hive.ssb_1g_csv.customer; 

--mv_3_2
CREATE MATERIALIZED VIEW mv_3_2
DISTRIBUTED BY HASH(lo_orderdate)
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES (
    "force_external_table_query_rewrite" = "TRUE"
)
AS
SELECT l.lo_orderdate, l.lo_orderkey, mv.c_custkey, mv.c_region, sum(l.lo_revenue)
FROM hive.ssb_1g_csv.lineorder l 
INNER JOIN mv_3_1 mv
ON l.lo_custkey = mv.c_custkey
GROUP BY l.lo_orderkey, l.lo_orderdate, mv.c_custkey, mv.c_region;
```

### 案例四：数据湖中实时数据和历史数据的冷热数据分离

考虑以下场景：过去三天内更新的数据直接写入 StarRocks，而较早的数据则经过检查并批量写入 Hive。然而，仍有查询可能涉及过去七天的数据。在这种情况下，您可以创建一个简单的模型，使用物化视图自动过期数据。

```SQL
CREATE MATERIALIZED VIEW mv_4_1 
DISTRIBUTED BY HASH(lo_orderdate)
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
AS 
SELECT lo_orderkey, lo_orderdate, lo_revenue
FROM hive.ssb_1g_csv.lineorder
WHERE lo_orderdate<=current_date()
AND lo_orderdate>=date_add(current_date(), INTERVAL -4 DAY);
```

您可以根据上层应用的逻辑，在此基础上进一步构建视图或物化视图。
```SQL
CREATE MATERIALIZED VIEW mv_4_1 
DISTRIBUTED BY HASH(lo_orderdate)
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
AS 
SELECT lo_orderkey, lo_orderdate, lo_revenue
FROM hive.ssb_1g_csv.lineorder
WHERE lo_orderdate<=current_date()
AND lo_orderdate>=date_add(current_date(), INTERVAL -4 DAY);
```

您可以根据上层应用的逻辑，在其上进一步构建视图或物化视图。