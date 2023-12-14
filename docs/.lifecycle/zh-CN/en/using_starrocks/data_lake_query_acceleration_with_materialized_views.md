---
displayed_sidebar: "English"
---

# 使用物化视图加速数据湖查询

本主题描述了如何使用StarRocks的异步物化视图来优化数据湖中的查询性能。

StarRocks提供开箱即用的数据湖查询能力，对于数据湖中的数据进行探索性查询和分析非常有效。在大多数情况下，[数据缓存](../data_source/data_cache.md)可以提供块级文件缓存，避免由远程存储抖动和大量I/O操作造成的性能下降。

但是，当需要构建复杂和高效的报表以及进一步加速这些查询时，您可能仍会遇到性能挑战。通过使用异步物化视图，您可以实现报表和数据应用在数据湖上的更高并发性和更好的查询性能。

## 概述

StarRocks支持基于外部目录（如Hive目录、Iceberg目录和Hudi目录）构建异步物化视图。外部目录基物化视图在以下情形下特别有用：

- **透明加速数据湖报表**

  为了确保数据湖报表的查询性能，数据工程师通常需要与数据分析师紧密合作，以探究报表加速层的构建逻辑。如果加速层需要进一步更新，它们必须相应更新构建逻辑、处理计划和查询语句。

  通过物化视图的查询重写能力，报表加速可以变得透明，对用户来说是不可察觉的。当慢查询被识别出时，数据工程师可以分析慢查询的模式并按需创建物化视图。然后，应用端查询会被智能地重写，并通过物化视图透明加速，从而实现查询性能的快速改进，而无需修改业务应用程序或查询语句的逻辑。

- **增量计算实时数据与历史数据关联**

  假设您的业务应用需要将StarRocks原生表中的实时数据与数据湖中的历史数据进行关联进行增量计算。在这种情况下，物化视图可以提供直接的解决方案。例如，如果实时事实表是StarRocks中的原生表，维度表存储在数据湖中，您可以通过构建将原生表与外部数据源中的表进行关联的物化视图来轻松进行增量计算。

- **指标层的快速构建**

  在处理高维数据时，计算和处理指标可能会遇到挑战。您可以使用物化视图来进行数据预聚合和汇总，创建相对轻量级的指标层。此外，物化视图可以自动刷新，进一步减少指标计算的复杂性。

物化视图、数据缓存和StarRocks中的原生表都是实现显著查询性能提升的有效方法。以下表格比较了它们的主要差异：

<table class="comparison">
  <thead>
    <tr>
      <th>&nbsp;</th>
      <th>数据缓存</th>
      <th>物化视图</th>
      <th>原生表</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>数据加载和更新</b></td>
      <td>查询会自动触发数据缓存。</td>
      <td>刷新任务会自动触发。</td>
      <td>支持各种导入方法，但需要手动维护导入任务。</td>
    </tr>
    <tr>
      <td><b>数据缓存粒度</b></td>
      <td><ul><li>支持块级数据缓存</li><li>遵循LRU缓存淘汰机制</li><li>不会缓存计算结果</li></ul></td>
      <td>存储预先计算的查询结果</td>
      <td>基于表结构存储数据</td>
    </tr>
    <tr>
      <td><b>查询性能</b></td>
      <td colspan="3" style={{textAlign: 'center'}} >数据缓存 ≤ 物化视图 = 原生表</td>
    </tr>
    <tr>
      <td><b>查询语句</b></td>
      <td><ul><li>无需修改针对数据湖的查询语句</li><li>一旦查询命中缓存，就会进行计算。</li></ul></td>
      <td><ul><li>无需修改针对数据湖的查询语句</li><li>利用查询重写重用预先计算的结果</li></ul></td>
      <td>需要修改查询语句以查询原生表</td>
    </tr>
  </tbody>
</table>

<br />

与直接查询数据湖数据或将数据加载到原生表中相比，物化视图提供了几个独特的优势：

- **本地存储加速**：物化视图可以利用StarRocks的本地存储加速优势，如索引、分区、分桶和共同存储组，从而比直接从数据湖查询数据获得更好的查询性能。
- **加载任务零维护**：物化视图通过自动刷新任务透明地更新数据。无需维护加载任务以执行计划中的数据更新。此外，基于Hive目录的物化视图可以检测数据更改并在分区级别进行增量刷新。
- **智能查询重写**：查询可以被透明地重写以使用物化视图。您可以立即受益于加速，无需修改应用程序使用的查询语句。

<br />

因此，我们建议在以下情况下使用物化视图：

- 即使启用了数据缓存，查询性能仍不满足查询延迟和并发性的要求。
- 查询涉及可重用组件，例如固定的聚合函数或连接模式。
- 数据以分区组织，而查询涉及相对较高级别的聚合（例如按天进行聚合）。

<br />

在以下情况下，我们建议优先使用数据缓存进行加速：

- 查询没有许多可重用的组件，并且可能扫描数据湖中的任何数据。
- 远程存储波动或不稳定，可能会影响访问。

## 创建基于外部目录的物化视图

在外部目录的表上创建物化视图与在StarRiocks的原生表上创建物化视图类似。您只需要根据您使用的数据源设置合适的刷新策略，并手动启用基于外部目录的物化视图的查询重写。

### 选择适当的刷新策略

当前，StarRocks无法检测Hudi目录和JDBC目录中分区级别的数据更改。因此，一旦触发任务，将执行全尺寸的刷新。

对于Hive目录和Iceberg目录（从v3.1.4开始），StarRocks支持在分区级别检测数据更改。因此，StarRocks可以：

- 仅刷新具有数据更改的分区，避免执行全尺寸刷新，降低由刷新引起的资源消耗。

- 在查询重写期间在一定程度上确保数据一致性。如果数据湖中的基表发生数据更改，查询将不会被重写以使用物化视图。

  > **注意**
  >
  > 您仍然可以通过设置属性`mv_rewrite_staleness_second`来容忍一定程度的数据不一致性，以创建物化视图。有关更多信息，请参阅[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)。

请注意，如果您需要按分区进行刷新，物化视图的分区键必须包含基表的分区键。

对于Hive目录，您可以启用Hive元数据缓存刷新功能，以允许StarRocks检测分区级别的数据更改。启用此功能后，StarRocks会定期访问Hive Metastore Service（HMS）或AWS Glue，检查最近查询的热数据的元数据信息。

要启用Hive元数据缓存刷新功能，您可以使用[ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)设置以下FE动态配置项：

| **配置项**                                              | **默认值**               | **描述**                                                     |
| ------------------------------------------------------- | ----------------------- | ------------------------------------------------------------ |
| enable_background_refresh_connector_metadata            | v3.0中为true v2.5中为false | 是否启用定期Hive元数据缓存刷新。启用后，StarRocks会轮询您的Hive集群的元数据存储（Hive Metastore或AWS Glue），并刷新缓存的频繁访问的Hive目录元数据信息。true表示启用Hive元数据缓存刷新，false表示禁用。 |
| background_refresh_metadata_interval_millis             | 600000（10分钟）           | 两次连续Hive元数据缓存刷新之间的间隔。单位：毫秒。 |
| background_refresh_metadata_time_secs_since_last_access_secs | 86400（24小时）           | Hive元数据缓存刷新任务的过期时间。对于已访问的Hive目录，如果它在指定时间内没有访问，StarRocks将停止刷新其缓存的元数据。对于尚未访问的Hive目录，StarRocks将不刷新其缓存的元数据。单位：秒。 |

从v3.1.4开始，StarRocks支持在Iceberg目录上按分区级别检测数据更改。目前仅支持Iceberg V1表。

### 为基于外部目录的物化视图启用查询重写


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

在涉及查询重写的场景中，如果您使用非常复杂的查询语句来构建物化视图，我们建议您将查询语句拆分并以嵌套方式构建多个简单的物化视图。嵌套物化视图更加灵活，可以适应更广泛的查询模式。


## 最佳实践

在真实的业务场景中，您可以通过分析审计日志或[big query logs](../administration/monitor_manage_big_queries.md#analyze-big-query-logs)来识别执行延迟和资源消耗较高的查询。您还可以使用[query profiles](../administration/query_profile.md)来准确定位查询执行缓慢的特定阶段。以下各节提供了有关如何使用物化视图提升数据湖查询性能的说明和示例。

### 案例一：加速数据湖中的连接计算

您可以使用物化视图来加速数据湖中的连接查询。

假设在您的Hive目录中，以下查询特别缓慢：

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

通过分析它们的查询配置文件，您可能会注意到查询执行时间主要花在表`lineorder`和其他维度表在列`lo_orderdate`上的哈希连接上。

在这里，Q1和Q2在`lineorder`和`dates`连接后执行聚合，而Q3在连接`lineorder`、`dates`、`part`和`supplier`后执行聚合。

因此，您可以利用StarRocks的[View Delta Join rewrite](./query_rewrite_with_materialized_views.md#view-delta-join-rewrite)功能，构建一个连接`lineorder`、`dates`、`part`和`supplier`的物化视图。

```SQL
CREATE MATERIALIZED VIEW lineorder_flat_mv
DISTRIBUTED BY HASH(LO_ORDERDATE, LO_ORDERKEY) BUCKETS 48
PARTITION BY LO_ORDERDATE
REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
PROPERTIES ( 
    -- 指定唯一约束。
    "unique_constraints" = "
    hive.ssb_1g_csv.supplier.s_suppkey;
    hive.ssb_1g_csv.part.p_partkey;
    hive.ssb_1g_csv.dates.d_datekey",
    -- 指定外键。
    "foreign_key_constraints" = "
    hive.ssb_1g_csv.lineorder(lo_partkey) REFERENCES hive.ssb_1g_csv.part(p_partkey);
    hive.ssb_1g_csv.lineorder(lo_suppkey) REFERENCES hive.ssb_1g_csv.supplier(s_suppkey);
    hive.ssb_1g_csv.lineorder(lo_orderdate) REFERENCES hive.ssb_1g_csv.dates(d_datekey)",
    -- 为外部目录基础的物化视图启用查询重写。
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

### 案例二：加速数据湖中的聚合和连接上的聚合

无论是在单个表上还是涉及多个表，都可以使用物化视图来加速聚合查询。

- 单表聚合查询

  对于典型的单表查询，它们的查询配置文件会显示AGGREGATE节点消耗了大量时间。您可以使用常见的聚合运算符构建物化视图。

  假设以下是一个缓慢的查询：

  ```SQL
  --Q4
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate
  ORDER BY lo_orderdate limit 100;
  ```

  Q4 计算了唯一订单的每日数量。因为计算唯一订单的数量可能是计算密集型的，您可以创建以下两种类型的物化视图来加速：

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
  -- lo_orderkey 必须是BIGINT类型，以便它可以用于查询重写。
  lo_orderdate, bitmap_union(to_bitmap(lo_orderkey))
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  ```

  请注意，在此上下文中，不要创建带有LIMIT和ORDER BY子句的物化视图，以避免重写失败。有关查询重写的限制的更多信息，请参阅[Query rewrite with materialized views - Limitations](./query_rewrite_with_materialized_views.md#limitations)。

- 多表聚合查询

  在涉及连接结果的聚合场景中，您可以在已有的连接表上创建嵌套物化视图，以进一步聚合连接结果。例如，在案例一中的示例基础上，您可以创建以下物化视图来加速 Q1 和 Q2，因为它们的聚合模式类似：

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

  当然，也有可能在单个物化视图中执行连接和聚合计算。尽管这些类型的物化视图可能有更少的查询重写机会（由于它们的特定计算），但它们通常在聚合后占用较少的存储空间。您的选择可以基于您的具体用例。

  ```SQL
  CREATE MATERIALIZED VIEW mv_2_4
  DISTRIBUTED BY HASH(lo_orderdate)
  PARTITION BY LO_ORDERDATE
  REFRESH ASYNC EVERY(INTERVAL 1 DAY) 
  PROPERTIES (
      "force_external_table_query_rewrite" = "TRUE"
  )
  AS
```SQL
    + {T}
    + {T}
+ {T}
  + {T}
```