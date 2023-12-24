---
displayed_sidebar: English
---

# 使用物化视图加速数据湖查询

本主题描述了如何使用 StarRocks 的异步物化视图来优化数据湖中的查询性能。

StarRocks 提供开箱即用的数据湖查询功能，非常适用于对数据湖中的数据进行探索性查询和分析。在大多数情况下，[数据缓存](../data_source/data_cache.md)可以提供块级文件缓存，避免由远程存储抖动和大量 I/O 操作导致的性能下降。

然而，当需要构建复杂而高效的报表或进一步加速这些查询时，您可能会遇到性能挑战。通过使用异步物化视图，您可以实现对数据湖中报表和数据应用程序的更高并发性和更好的查询性能。

## 概述

StarRocks 支持基于外部目录（如 Hive 目录、Iceberg 目录和 Hudi 目录）构建异步物化视图。基于外部目录的物化视图在以下情况下特别有用：

- **数据湖报表的透明加速**

  为了保证数据湖报表的查询性能，数据工程师通常需要与数据分析师紧密合作，探究报表加速层的构建逻辑。如果加速层需要进一步更新，则需要相应地更新构造逻辑、处理调度和查询语句。

  通过物化视图的查询重写能力，可以使报表加速透明化，用户无法察觉。当识别出慢查询时，数据工程师可以分析慢查询的模式，并按需创建物化视图。然后，通过物化视图智能地重写和透明地加速应用程序端查询，从而在不修改业务应用程序或查询语句的逻辑的情况下快速提高查询性能。

- **与历史数据关联的实时数据的增量计算**

  假设您的业务应用需要关联 StarRocks 原生表中的实时数据和数据湖中的历史数据进行增量计算。在这种情况下，物化视图可以提供简单的解决方案。例如，如果实时事实表是 StarRocks 中的原生表，维度表存储在数据湖中，则通过构建物化视图，将原生表与外部数据源中的表关联起来，可以方便地进行增量计算。

- **快速构建公制层**

  在处理高维数据时，计算和处理指标可能会遇到挑战。您可以使用物化视图（允许您执行数据预聚合和汇总）来创建相对轻量级的指标层。此外，物化视图可以自动刷新，进一步降低了指标计算的复杂性。

物化视图、数据缓存和 StarRocks 中的原生表都是显著提升查询性能的有效方法。下表比较了它们的主要区别：

<table class="comparison">
  <thead>
    <tr>
      <th>&nbsp;</th>
      <th>数据缓存</th>
      <th>物化视图</th>
      <th>本机表</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>数据加载和更新</b></td>
      <td>查询会自动触发数据缓存。</td>
      <td>刷新任务是自动触发的。</td>
      <td>支持多种导入方式，但需要手动维护导入任务</td>
    </tr>
    <tr>
      <td><b>数据缓存粒度</b></td>
      <td><ul><li>支持块级数据缓存</li><li>遵循 LRU 缓存逐出机制</li><li>不缓存任何计算结果</li></ul></td>
      <td>存储预先计算的查询结果</td>
      <td>基于表架构存储数据</td>
    </tr>
    <tr>
      <td><b>查询性能</b></td>
      <td colspan="3" style={{textAlign: 'center'}} >数据缓存 &le; 物化视图 = 本机表</td>
    </tr>
    <tr>
      <td><b>查询语句</b></td>
      <td><ul><li>无需修改针对数据湖的查询语句</li><li>一旦查询命中缓存，就会发生计算。</li></ul></td>
      <td><ul><li>无需修改针对数据湖的查询语句</li><li>利用查询重写来重用预先计算的结果</li></ul></td>
      <td>需要修改查询语句来查询本机表</td>
    </tr>
  </tbody>
</table>

<br />

与直接查询湖数据或将数据加载到本机表中相比，物化视图具有以下几个独特优势：

- **本地存储加速**：物化视图可以充分利用 StarRocks 在本地存储方面的加速优势，例如索引、分区、存储桶和并置组，与直接从数据湖查询数据相比，查询性能更好。
- **加载任务零维护**：物化视图通过自动刷新任务透明地更新数据。无需维护加载任务即可执行计划的数据更新。此外，基于 Hive 目录的物化视图可以检测数据更改，并在分区级别执行增量刷新。
- **智能查询重写**：可以透明地重写查询以使用物化视图。您可以立即从加速中受益，而无需修改应用程序使用的查询语句。

<br />

因此，我们建议在以下场景中使用物化视图：

- 即使启用了数据缓存，查询性能也无法满足查询延迟和并发性要求。
- 查询涉及可重用的组件，例如固定的聚合函数或联接模式。
- 数据按分区进行组织，而查询涉及相对较高级别的聚合（例如，按天聚合）。

<br />

在以下场景中，我们建议优先通过数据缓存加速：

- 查询没有许多可重用的组件，可以扫描数据湖中的任何数据。
- 远程存储存在显著波动或不稳定，这可能会影响访问。

## 创建基于外部目录的物化视图

在外部目录中的表上创建物化视图类似于在 StarRiocks 的本机表上创建物化视图。您只需根据所使用的数据源设置合适的刷新策略，并手动为基于目录的外部物化视图启用查询重写。

### 选择合适的刷新策略

目前，StarRocks 无法检测到 Hudi 目录和 JDBC 目录的分区级数据变化。因此，一旦触发任务，就会执行全尺寸刷新。

对于 Hive Catalog 和 Iceberg Catalog（从 v3.1.4 开始），StarRocks 支持在分区级别检测数据变化。因此，StarRocks 可以：

- 仅刷新有数据更改的分区，避免全尺寸刷新，减少刷新导致的资源消耗。

- 在查询重写过程中，在一定程度上保证数据的一致性。如果数据湖中的基表中存在数据更改，则不会重写查询以使用物化视图。

  > **注意**
  >
  > 您仍可以通过在创建物化视图时设置属性来选择容忍一定程度的数据不一致 `mv_rewrite_staleness_second` 。有关详细信息，请参阅 [CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)。

需要注意的是，如果需要按分区刷新，物化视图的分区键必须包含在基表的分区键中。

对于 Hive 目录，您可以开启 Hive 元数据缓存刷新功能，让 StarRocks 能够检测分区级别的数据变化。开启该功能后，StarRocks 会定期访问 Hive Metastore Service（HMS）或 AWS Glue，查看最近查询到的热点数据的元数据信息。

如需开启 Hive 元数据缓存刷新功能，可以使用 [ADMIN SET FRONTEND CONFIG 设置以下 FE 动态配置项](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)：

| **配置项**                                       | **违约**                | **描述**                                              |
| ------------------------------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| enable_background_refresh_connector_metadata                 | 在 v3.0 中为 true，在 v2.5 中为 false | 是否开启定期 Hive 元数据缓存刷新。开启后，StarRocks 会轮询 Hive 集群的元存储（Hive Metastore 或 AWS Glue），并刷新频繁访问的 Hive 目录的缓存元数据，以感知数据变化。true 表示启用 Hive 元数据缓存刷新，false 表示禁用刷新。 |
| background_refresh_metadata_interval_millis                  | 600000（10分钟）        | 两次连续 Hive 元数据缓存刷新之间的时间间隔。单位：毫秒。 |
| background_refresh_metadata_time_secs_since_last_access_secs | 86400（24小时）           | Hive 元数据缓存刷新任务的过期时间。对于已访问的 Hive 目录，如果超过指定时间未访问，StarRocks 将停止刷新其缓存的元数据。对于尚未访问的 Hive 目录，StarRocks 不会刷新其缓存的元数据。单位：秒。 |

从 v3.1.4 开始，StarRocks 支持在分区级别检测 Iceberg Catalog 的数据变化。目前仅支持 Iceberg V1 表。

### 为基于外部目录的物化视图启用查询重写

默认情况下，StarRocks 不支持对基于 Hudi、Iceberg 和 JDBC 目录构建的物化视图进行查询重写，因为该场景下的查询重写无法保证结果的强一致性。您可以通过在创建物化视图时将属性`force_external_table_query_rewrite`设置为`true`来启用此功能。对于基于 Hive 目录中的表构建的物化视图，默认情况下会启用查询重写。

例：

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


在涉及查询重写的场景下，如果您使用非常复杂的查询语句构建物化视图，建议拆分查询语句，并以嵌套方式构造多个简单的物化视图。嵌套的物化视图更加灵活，可以适应更广泛的查询模式。

## 最佳实践

在实际业务场景中，您可以通过分析审计日志或[大查询日志](../administration/monitor_manage_big_queries.md#analyze-big-query-logs)来识别执行延迟和资源消耗较高的查询。您还可以使用[查询配置文件](../administration/query_profile.md)来查明查询速度较慢的具体阶段。以下部分提供了有关如何使用物化视图提升数据湖查询性能的说明和示例。

### 案例一：加速数据湖中的联接计算

您可以使用物化视图来加速数据湖中的联接查询。

假设 Hive 目录上的以下查询速度特别慢：

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

通过分析它们的查询配置文件，您可能会注意到查询执行时间主要花在表`lineorder`和其他维度表的`lo_orderdate`列之间的哈希联接上。

其中，Q1 和 Q2 在`lineorder`和`dates`加入后执行聚合，而 Q3 在`lineorder`、`dates`、`part`和`supplier`加入后执行聚合。

因此，您可以利用 StarRocks 的[View Delta Join 重写](./query_rewrite_with_materialized_views.md#view-delta-join-rewrite)能力，构建一个物化视图，该物化视图加入了`lineorder`、`dates`、`part`和`supplier`。

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
    -- 启用外部基于目录的物化视图的查询重写。
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

### 案例二：加速数据湖中联接的聚合和聚合

物化视图可用于加速聚合查询，无论是在单个表上还是涉及多个表。

- 单表聚合查询

  对于典型的单表查询，其查询配置文件将显示 AGGREGATE 节点消耗了大量时间。您可以使用常见的聚合运算符来构造物化视图。

  假设以下查询速度较慢：

  ```SQL
  --Q4
  SELECT
  lo_orderdate, count(distinct lo_orderkey)
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate
  ORDER BY lo_orderdate limit 100;
  ```

  Q4 计算每日唯一订单数。由于计算不重复计数可能成本很高，因此您可以创建以下两种类型的物化视图来加速：

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
  -- lo_orderkey 必须是 BIGINT 类型，以便用于查询重写。
  lo_orderdate, bitmap_union(to_bitmap(lo_orderkey))
  FROM hive.ssb_1g_csv.lineorder
  GROUP BY lo_orderdate;
  ```

  请注意，在此上下文中，请勿使用 LIMIT 和 ORDER BY 子句创建物化视图，以避免重写失败。有关查询重写限制的详细信息，请参阅[使用物化视图进行查询重写 - 限制](./query_rewrite_with_materialized_views.md#limitations)。

- 多表聚合查询

  在涉及联接结果聚合的方案中，可以在现有联接表的物化视图上创建嵌套的物化视图，以进一步聚合联接结果。例如，基于案例 1 的示例，您可以创建以下物化视图来加速 Q1 和 Q2，因为它们的聚合模式相似：

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

  当然，也可以在单个物化视图中执行联接和聚合计算。虽然这些类型的物化视图可能具有较少的查询重写机会（由于其特定的计算），但它们在聚合后通常占用较少的存储空间。您的选择可以基于您的具体用例。

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

### 案例三：在数据湖中加速联接的聚合

在某些情况下，可能需要先对一个表执行聚合计算，然后再对其他表执行联接查询。为了充分利用 StarRocks 的查询重写能力，我们建议构建嵌套的物化视图。例如：

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

Q5 首先对`customer`表执行聚合查询，然后对`lineorder`表执行联接和聚合。类似的查询可能涉及`c_region`和`lo_orderdate`上的不同筛选器。若要利用查询重写功能，可以创建两个物化视图，一个用于聚合，另一个用于联接。

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

### 案例四：在数据湖中将实时数据和历史数据分离

考虑以下场景：过去三天内更新的数据直接写入 StarRocks，而不太新的数据则被检查并批量写入 Hive。但是，仍有查询可能涉及过去 7 天的数据。在这种情况下，您可以创建一个简单的模型，使用物化视图自动使数据过期。

```SQL

创建物化视图 mv_4_1 
分布方式 HASH(lo_orderdate)
按 LO_ORDERDATE 分区
异步刷新 每(间隔 1 天) 
作为 
选择 lo_orderkey, lo_orderdate, lo_revenue
从 hive.ssb_1g_csv.lineorder
其中 lo_orderdate<=current_date()
并且 lo_orderdate>=date_add(current_date(), 间隔 -4 天);
```

您可以根据上层应用的逻辑，基于此进一步构建视图或物化视图。