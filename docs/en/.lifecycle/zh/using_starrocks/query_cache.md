---
displayed_sidebar: English
---

# 查询缓存

查询缓存是 StarRocks 的一个强大功能，可以极大地增强聚合查询的性能。通过将本地聚合的中间结果存储在内存中，查询缓存可以避免不必要的磁盘访问和计算，从而加速新的与之前相同或相似的查询。有了查询缓存，StarRocks 可以为聚合查询提供快速准确的结果，节省时间和资源，并实现更好的可扩展性。查询缓存在许多用户在大型复杂数据集上运行类似查询的高并发场景中特别有用。

此功能从 v2.5 版本开始支持。

在 v2.5 版本中，查询缓存仅支持对单个平面表的聚合查询。从 v3.0 版本开始，查询缓存还支持对以星型架构联接的多个表进行聚合查询。

## 应用场景

我们建议您在以下场景中使用查询缓存：

- 您经常对单个平面表或在星型架构中连接的多个联接表运行聚合查询。
- 大多数聚合查询是非 GROUP BY 聚合查询和低基数 GROUP BY 聚合查询。
- 您的数据是按时间分区以追加方式加载的，可以根据访问频率分为热数据和冷数据。

查询缓存支持满足以下条件的查询：

- 查询引擎为 Pipeline。要启用 Pipeline 引擎，请将会话变量 `enable_pipeline_engine` 设置为 `true`。

  > **注意**
  >
  > 其他查询引擎不支持查询缓存。

- 查询位于本机 OLAP 表（从 v2.5 版本开始）或云原生表（从 v3.0 版本开始）上。查询缓存不支持对外部表的查询。查询缓存还支持其计划需要访问同步实例化视图的查询。但是，查询缓存不支持其计划需要访问异步实例化视图的查询。

- 查询为单个表或多个联接表的聚合查询。

  **注意**
  >
  > - 查询缓存支持广播联接和桶洗牌联接。
  > - 查询缓存支持包含联接运算符的两个树结构：聚合-联接和联接-聚合。聚合-联接树结构不支持洗牌联接，而联接-聚合树结构不支持哈希联接。

- 查询不包括 `rand`、`random`、`uuid` 和 `sleep` 等非确定性函数。

查询缓存支持使用以下任何分区策略的表进行查询：未分区、多列分区和单列分区。

## 功能边界

- 查询缓存基于 Pipeline 引擎的每个平板电脑的计算。每个平板电脑的计算意味着管道驱动程序可以逐个处理整个平板电脑，而不是处理部分或交错的多个平板电脑。如果每个单独的 BE 需要处理的平板电脑数大于或等于为运行此查询而调用的管道驱动程序数，则查询缓存将起作用。调用的管道驱动程序数表示实际并行度（DOP）。如果平板电脑的数量小于管道驱动程序的数量，则每个管道驱动程序仅处理特定平板电脑的一部分。在此情况下，无法生成每个平板电脑的计算结果，因此查询缓存不起作用。
- 在 StarRocks 中，聚合查询至少包含四个阶段。只有当 OlapScanNode 和 AggregateNode 从同一 Fragment 计算数据时，才能缓存第一阶段 AggregateNode 生成的 Per-Tablet 计算结果。AggregateNode 在其他阶段生成的 Per-Tablet 计算结果无法缓存。对于某些 DISTINCT 聚合查询，如果会话变量 `cbo_cte_reuse` 设置为 `true`，则当生成数据的 OlapScanNode 和使用生成的数据的 stage-1 AggregateNode 从不同片段计算数据并由 ExchangeNode 桥接时，查询缓存不起作用。以下两个示例显示了执行 CTE 优化，因此查询缓存不起作用的方案：
  - 输出列是使用 `avg(distinct)` 等聚合函数计算的。
  - 输出列由多个 DISTINCT 聚合函数计算。
- 如果数据在聚合之前被随机排序，则查询缓存无法加速对该数据的查询。
- 如果表的分组依据列或重复数据删除列是高基数列，则将为该表的聚合查询生成较大的结果。在这些情况下，查询将在运行时绕过查询缓存。
- 查询缓存占用 BE 提供的少量内存来保存计算结果。查询缓存的大小默认为 512 MB。因此，查询缓存不适合保存大型数据项。此外，启用查询缓存后，如果缓存命中率较低，则查询性能会降低。因此，如果为平板电脑生成的计算结果的大小超过 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 参数指定的阈值，则查询缓存将不再适用于该查询，并且查询将切换到直通模式。

## 工作原理

启用查询缓存后，每个 BE 将查询的本地聚合拆分为以下两个阶段：

1. 每片机聚合

   BE 单独处理每片药片。当 BE 开始处理平板电脑时，它首先探测查询缓存，以查看该平板电脑上聚合的中间结果是否在查询缓存中。如果是这样（缓存命中），BE 将直接从查询缓存中获取中间结果。如果没有（缓存未命中），则 BE 访问磁盘上的数据并执行本地聚合以计算中间结果。当 BE 处理完平板电脑时，它会使用该平板电脑上聚合的中间结果填充查询缓存。

2. 平板电脑间聚合

   BE 从查询中涉及的所有平板电脑收集中间结果，并将它们合并为最终结果。

   ![查询缓存 - 工作原理 - 1](../assets/query_cache_principle-1.png)

将来发出类似查询时，它可以将缓存的结果重用到上一个查询中。例如，下图中显示的查询涉及三个平板电脑（平板电脑 0 到 2），并且第一个平板电脑（平板电脑 0）的中间结果已在查询缓存中。在此示例中，BE 可以直接从查询缓存中获取 Tablet 0 的结果，而不是访问磁盘上的数据。如果查询缓存已完全预热，它可以包含所有三个平板电脑的中间结果，因此 BE 不需要访问磁盘上的任何数据。

![查询缓存 - 工作原理 - 2](../assets/query_cache_principle-2.png)

为了释放额外的内存，查询缓存采用基于最近最少使用（LRU）的逐出策略来管理其中的缓存条目。根据此逐出策略，当查询缓存占用的内存量超过其预定义的大小 (`query_cache_capacity`) 时，最近使用最少的缓存条目将从查询缓存中逐出。

> **注意**
>
> 未来，StarRocks 还将支持基于生存时间（Time to Live，TTL）的逐出策略，该策略可以将缓存条目从查询缓存中逐出。

FE 通过查询缓存来判断是否需要对每个查询进行加速，并对查询进行规范化处理，以消除对查询语义没有影响的琐碎文字细节。

为了防止查询缓存的不良情况导致性能损失，BE 采用自适应策略在运行时绕过查询缓存。

## 启用查询缓存

本节介绍用于启用和配置查询缓存的参数和会话变量。

### FE 会话变量

| **变量**                | **默认值** | **可动态配置** | **描述**                                              |
| --------------------------- | ----------------- | --------------------------------- | ------------------------------------------------------------ |
| enable_query_cache          | false             | 是                               | 指定是否启用查询缓存。有效值： `true` 和 `false`。 `true` 指定启用此功能，`false` 指定禁用此功能。启用查询缓存后，它仅适用于满足本主题的“[应用场景](../using_starrocks/query_cache.md#application-scenarios)”一节中指定的条件的查询。 |
| query_cache_entry_max_bytes | 4194304           | 是                               | 指定触发直通模式的阈值。有效值： `0` 到 `9223372036854775807`。当查询访问的特定平板计算结果的字节数或行数超过 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 参数指定的阈值时，查询将切换为直通模式。<br />如果 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 参数设置为 `0`，即使所涉及的平板电脑未生成任何计算结果，也会使用直通模式。 |
| query_cache_entry_max_rows  | 409600            | 是                               | 同上。                                                |

### BE 参数

您需要在 BE 配置文件 **be.conf** 中配置以下参数。为 BE 重新配置该参数后，必须重启 BE 才能使新的参数设置生效。

| **参数**        | **必填** | **描述**                                              |
| -------------------- | ------------ | ------------------------------------------------------------ |
| query_cache_capacity | 否           | 指定查询缓存的大小。单位：字节。默认大小为 512 MB。<br />每个 BE 在内存中都有自己的本地查询缓存，并且它只填充和探测自己的查询缓存。<br />请注意，查询缓存大小不能小于 4 MB。如果 BE 的内存容量不足以预置预期的查询缓存大小，则可以增加 BE 的内存容量。 |

## 专为在所有场景中实现最大缓存命中率而设计

请考虑三种情况，即使查询在字面上不完全相同，查询缓存仍然有效。这三种情况是：

- 语义上等价的查询
- 具有重叠扫描分区的查询
- 针对仅追加数据更改的数据进行查询（无 UPDATE 或 DELETE 操作）

### 语义等效查询

当两个查询相似时，并不意味着它们必须从字面上等效，而是意味着它们的执行计划中包含语义等效的片段，它们被认为是语义等效的，并且可以重用彼此的计算结果。从广义上讲，如果两个查询从同一源查询数据，使用相同的计算方法，并且具有相同的执行计划，则它们在语义上是等价的。StarRocks 使用以下规则来评估两个查询在语义上是否等效：

- 如果这两个查询包含多个聚合，则只要它们的第一个聚合在语义上等效，它们就会被评估为语义等效。例如，以下两个查询 Q1 和 Q2 都包含多个聚合，但它们的第一个聚合在语义上是等效的。因此，Q1 和 Q2 在语义上被评估为等价。

  - Q1

    ```SQL
    SELECT
        (
            ifnull(sum(murmur_hash3_32(hour)), 0) + ifnull(sum(murmur_hash3_32(k0)), 0) + ifnull(sum(murmur_hash3_32(__c_0)), 0)
        ) AS fingerprint
    FROM
        (
            SELECT
                date_trunc('hour', ts) AS hour,
                k0,
                sum(v1) AS __c_0
            FROM
                t0
            WHERE
                ts between '2022-01-03 00:00:00'
                and '2022-01-03 23:59:59'
            GROUP BY
                date_trunc('hour', ts),
                k0
        ) AS t;
    ```

  - Q2

    ```SQL
    SELECT
        date_trunc('hour', ts) AS hour,
        k0,
        sum(v1) AS __c_0
    FROM
        t0
    WHERE
        ts between '2022-01-03 00:00:00'
        and '2022-01-03 23:59:59'
    GROUP BY
        date_trunc('hour', ts),
        k0
    ```

- 如果这两个查询都属于以下查询类型之一，则可以将它们评估为语义等效。请注意，包含 HAVING 子句的查询不能被评估为在语义上等同于不包含 HAVING 子句的查询。但是，包含 ORDER BY 或 LIMIT 子句不会影响对两个查询在语义上是否等效的评估。

  - GROUP BY 聚合

    ```SQL
    SELECT <GroupByItems>, <AggFunctionItems> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    [HAVING <HavingPredicate>] 
    ```

    > **注意**
    >
    > 在前面的示例中，HAVING 子句是可选的。

  - GROUP BY DISTINCT 聚合

    ```SQL
    SELECT DISTINCT <GroupByItems>, <Items> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    HAVING <HavingPredicate>
    ```

    > **注意**
    >
    > 在前面的示例中，HAVING 子句是可选的。

  - 非 GROUP BY 聚合

    ```SQL
    SELECT <AggFunctionItems> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

  - 非 GROUP BY DISTINCT 聚合

    ```SQL
    SELECT DISTINCT <Items> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

- 如果任一查询包含 `PartitionColumnRangePredicate`， `PartitionColumnRangePredicate` 则在评估这两个查询的语义等效性之前将其删除。 `PartitionColumnRangePredicate` 指定引用分区列的下列谓词类型之一：

  - `col between v1 and v2`：分区列的值在 [v1、v2] 和 `v1` 为常量表达式`v2`的范围内。
  - `v1 < col and col < v2`：分区列的值在 （v1， v2） 范围内，其中 `v1` 和 `v2` 是常量表达式。
  - `v1 < col and col <= v2`：分区列的值在 （v1， v2） 范围内，其中 `v1` 和 `v2` 是常量表达式。
  - `v1 <= col and col < v2`：分区列的值在 [v1， v2） 范围内，其中 `v1` 和 `v2` 是常量表达式。
  - `v1 <= col and col <= v2`：分区列的值在 [v1、v2] 和 `v1` 为常量表达式`v2`的范围内。

- 如果两个查询的 SELECT 子句的输出列在重新排列后相同，则这两个查询的计算结果为语义等效。

- 如果两个查询的 GROUP BY 子句的输出列在重新排列后相同，则这两个查询的计算结果为语义等效。

- 如果删除两个查询的 WHERE 子句的其余谓词在语义上等效，则`PartitionColumnRangePredicate`这两个查询的计算结果为语义等效。

- 如果两个查询的 HAVING 子句中的谓词在语义上是等价的，则这两个查询的计算结果为语义等效。

使用以下表 `lineorder_flat` 作为示例：

```SQL
CREATE TABLE `lineorder_flat`
(
    `lo_orderdate` date NOT NULL COMMENT "",
    `lo_orderkey` int(11) NOT NULL COMMENT "",
    `lo_linenumber` tinyint(4) NOT NULL COMMENT "",
    `lo_custkey` int(11) NOT NULL COMMENT "",
    `lo_partkey` int(11) NOT NULL COMMENT "",
    `lo_suppkey` int(11) NOT NULL COMMENT "",
    `lo_orderpriority` varchar(100) NOT NULL COMMENT "",
    `lo_shippriority` tinyint(4) NOT NULL COMMENT "",
    `lo_quantity` tinyint(4) NOT NULL COMMENT "",
    `lo_extendedprice` int(11) NOT NULL COMMENT "",
    `lo_ordtotalprice` int(11) NOT NULL COMMENT "",
    `lo_discount` tinyint(4) NOT NULL COMMENT "",
    `lo_revenue` int(11) NOT NULL COMMENT "",
    `lo_supplycost` int(11) NOT NULL COMMENT "",
    `lo_tax` tinyint(4) NOT NULL COMMENT "",
    `lo_commitdate` date NOT NULL COMMENT "",
    `lo_shipmode` varchar(100) NOT NULL COMMENT "",
    `c_name` varchar(100) NOT NULL COMMENT "",
    `c_address` varchar(100) NOT NULL COMMENT "",
    `c_city` varchar(100) NOT NULL COMMENT "",
    `c_nation` varchar(100) NOT NULL COMMENT "",
    `c_region` varchar(100) NOT NULL COMMENT "",
    `c_phone` varchar(100) NOT NULL COMMENT "",
    `c_mktsegment` varchar(100) NOT NULL COMMENT "",
    `s_name` varchar(100) NOT NULL COMMENT "",
    `s_address` varchar(100) NOT NULL COMMENT "",
    `s_city` varchar(100) NOT NULL COMMENT "",
    `s_nation` varchar(100) NOT NULL COMMENT "",
    `s_region` varchar(100) NOT NULL COMMENT "",
    `s_phone` varchar(100) NOT NULL COMMENT "",
    `p_name` varchar(100) NOT NULL COMMENT "",
    `p_mfgr` varchar(100) NOT NULL COMMENT "",
    `p_category` varchar(100) NOT NULL COMMENT "",
    `p_brand` varchar(100) NOT NULL COMMENT "",
    `p_color` varchar(100) NOT NULL COMMENT "",
    `p_type` varchar(100) NOT NULL COMMENT "",
    `p_size` tinyint(4) NOT NULL COMMENT "",
    `p_container` varchar(100) NOT NULL COMMENT ""
)
ENGINE=OLAP 
DUPLICATE KEY(`lo_orderdate`, `lo_orderkey`)
COMMENT "olap"
PARTITION BY RANGE(`lo_orderdate`)
(PARTITION p1 VALUES [('0000-01-01'), ('1993-01-01')),
PARTITION p2 VALUES [('1993-01-01'), ('1994-01-01')),
PARTITION p3 VALUES [('1994-01-01'), ('1995-01-01')),
PARTITION p4 VALUES [('1995-01-01'), ('1996-01-01')),
PARTITION p5 VALUES [('1996-01-01'), ('1997-01-01')),
PARTITION p6 VALUES [('1997-01-01'), ('1998-01-01')),
PARTITION p7 VALUES [('1998-01-01'), ('1999-01-01')))
DISTRIBUTED BY HASH(`lo_orderkey`)
PROPERTIES 
(
    "replication_num" = "1",
    "colocate_with" = "groupxx1",
    "storage_format" = "DEFAULT",
    "enable_persistent_index" = "false",
    "compression" = "LZ4"
);
```

表 `lineorder_flat` 上的以下两个查询 Q1 和 Q2 在经过以下处理后在语义上是等效的：

1. 重新排列 SELECT 语句的输出列。
2. 重新排列 GROUP BY 子句的输出列。
3. 删除 ORDER BY 子句的输出列。
4. 重新排列 WHERE 子句中的谓词。
5. 添加 `PartitionColumnRangePredicate`.

- Q1

  ```SQL
  SELECT sum(lo_revenue)), year(lo_orderdate) AS year,p_brand
  FROM lineorder_flat
  WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA'
  GROUP BY year,p_brand
  ORDER BY year,p_brand;
  ```

- Q2

  ```SQL
  SELECT year(lo_orderdate) AS year, p_brand, sum(lo_revenue))
  FROM lineorder_flat
  WHERE s_region = 'AMERICA' AND p_category = 'MFGR#12' AND 
     lo_orderdate >= '1993-01-01' AND lo_orderdate <= '1993-12-31'
  GROUP BY p_brand, year(lo_orderdate)
  ```

语义等效性是根据查询的物理计划来评估的。因此，查询中的文字差异不会影响语义等效性的评估。此外，常量表达式将从查询中删除，表达式 `cast` 也会在查询优化期间删除。因此，这些表达式不会影响语义等价的评估。第三，列和关系的别名也不会影响语义等价的评估。

### 具有重叠扫描分区的查询

查询缓存支持基于谓词的查询拆分。

基于谓词语义拆分查询有助于实现部分计算结果的重用。当查询中包含引用表分区列的谓词，且该谓词指定了取值范围时，StarRocks 可以根据表的分区情况，将该范围拆分为多个区间。每个单独间隔的计算结果可以单独由其他查询重用。

使用以下表 `t0` 作为示例：

```SQL
CREATE TABLE if not exists t0
(
    ts DATETIME NOT NULL,
    k0 VARCHAR(10) NOT NULL,
    k1 BIGINT NOT NULL,
    v1 DECIMAL64(7, 2) NOT NULL 
)
ENGINE=OLAP
DUPLICATE KEY(`ts`, `k0`, `k1`)
COMMENT "OLAP"
PARTITION BY RANGE(ts)
(
  START ("2022-01-01 00:00:00") END ("2022-02-01 00:00:00") EVERY (INTERVAL 1 day) 
)
DISTRIBUTED BY HASH(`ts`, `k0`, `k1`)
PROPERTIES
(
    "replication_num" = "1", 
    "storage_format" = "default"
);
表 `t0` 是按天分区的，列 `ts` 是表的分区列。在以下四个查询中，Q2、Q3 和 Q4 可以重用部分计算结果，这些结果是为 Q1 缓存的：

- Q1

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-14 23:59:59'
  GROUP BY day;
  ```

  Q1 中谓词 `ts between '2022-01-02 12:30:00' and '2022-01-14 23:59:59'` 指定的数值范围可以分割为以下区间：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  12. [2022-01-13 00:00:00, 2022-01-14 00:00:00),
  13. [2022-01-14 00:00:00, 2022-01-15 00:00:00),
  ```

- Q2

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-02 12:30:00' AND  ts < '2022-01-05 00:00:00'
  GROUP BY day;
  ```

  Q2 可以重用 Q1 中以下时间间隔的计算结果：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ```

- Q3

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-01 12:30:00' AND  ts <= '2022-01-10 12:00:00'
  GROUP BY day;
  ```

  Q3 可以重用 Q1 中以下时间间隔的计算结果：

  ```SQL
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  8. [2022-01-09 00:00:00, 2022-01-10 00:00:00),
  ```

- Q4

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' and '2022-01-02 23:59:59'
  GROUP BY day;
  ```

  Q4 可以重用 Q1 中以下时间间隔的计算结果：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  ```

对部分计算结果的重用支持因使用的分区策略而异，如下表所示。

| **分区策略**         | **支持重用部分计算结果**         |
| ------------------- | ------------------------------------------------------------ |
| 未分区             | 不支持                                                |
| 多列分区           | 不支持<br />**注意**<br />此功能可能会在将来得到支持。 |
| 单列分区           | 支持                                                    |

### 针对仅追加数据更改的数据进行查询

查询缓存支持多版本缓存。

随着数据加载的进行，会生成新版本的 tablet。因此，从以前版本的 tablet 生成的缓存计算结果会变得陈旧，并且滞后于最新的 tablet 版本。在这种情况下，多版本缓存机制会尝试将保存在查询缓存中的过时结果和存储在磁盘上的 tablet 的增量版本合并到 tablet 的最终结果中，以便新查询可以携带最新的 tablet 版本。多版本缓存受表类型、查询类型和数据更新类型的约束。

对多版本缓存的支持因表类型和查询类型而异，如下表所示。

| **表类型**           | **查询类型**                                           | **支持多版本缓存**                        |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 重复键表           | <ul><li>基表的查询</li><li>同步实例化视图的查询</li></ul> | <ul><li>基表查询：在除增量 tablet 版本包含数据删除记录的所有情况下均受支持。</li><li>同步实例化视图的查询：在所有情况下均受支持，但查询的 GROUP BY、HAVING 或 WHERE 子句引用聚合列时除外。</li></ul> |
| 聚合表             | 基表的查询或同步实例化视图的查询                         | 在所有情况下均受支持，但不支持以下情况：基表的架构包含聚合函数 `replace`。查询的 GROUP BY、HAVING 或 WHERE 子句引用聚合列。增量 tablet 版本包含数据删除记录。 |
| 唯一键表           | 不适用                                                | 不支持。但是，支持查询缓存。        |
| 主键表             | 不适用                                                | 不支持。但是，支持查询缓存。        |

数据更新类型对多版本缓存的影响如下：

- 数据删除

  如果 tablet 的增量版本包含删除操作，则多版本缓存无法工作。

- 数据插入

  - 如果为 tablet 生成空版本，则查询缓存中 tablet 的现有数据仍然有效，并且仍然可以检索。
  - 如果为 tablet 生成了非空版本，则查询缓存中 tablet 的现有数据仍然有效，但其版本滞后于最新版本的 tablet。在这种情况下，StarRocks 会将现有数据版本生成的增量数据读取到最新版本的 tablet 中，并将现有数据与增量数据进行合并，并将合并后的数据填充到查询缓存中。

- 架构更改和 tablet 截断

  如果表的架构发生更改或表的特定 tablet 被截断，则会为该表生成新的 tablet。因此，查询缓存中表的 tablet 的现有数据将失效。

## 指标

查询缓存所针对的查询配置文件包含 `CacheOperator` 统计信息。

在查询的源计划中，如果管道包含 `OlapScanOperator`，则 `OlapScanOperator` 和聚合运算符的名称会以 `ML_` 为前缀，以表示管道使用 `MultilaneOperator` 执行每个 tablet 的计算。 `CacheOperator` 被插入到 `ML_CONJUGATE_AGGREGATE` 之前，用于处理控制查询缓存在直通、填充和探测模式下运行方式的逻辑。查询的配置文件包含以下 `CacheOperator` 指标，可帮助您了解查询缓存使用情况。

| **度量**                | **描述**                                              |
| ------------------------- | ------------------------------------------------------------ |
| 缓存传递字节     | 在直通模式下生成的字节数。           |
| 缓存传递块数  | 在直通模式下生成的块数。          |
| 缓存传递行数    | 在直通模式下生成的行数。            |
| 缓存传递平板电脑数 | 在直通模式下生成的平板电脑数量。         |
| CachePassthroughTime：     | 直通模式下花费的计算时间量。    |
| 缓存填充字节        | 在填充模式下生成的字节数。              |
| 缓存填充块数     | 在填充模式下生成的块数。             |
| 缓存填充行数       | 在填充模式下生成的行数。               |
| 缓存填充平板电脑数    | 在填充模式下生成的平板电脑数。            |
| CachePopulateTime（缓存填充时间）         | 填充模式下花费的计算时间量。       |
| CacheProbeBytes           | 在探测模式下为缓存命中生成的字节数。  |
| 缓存探测块数        | 在探测模式下为缓存命中生成的块数。 |
| CacheProbeRowNum          | 在探测模式下为缓存命中生成的行数。   |
| CacheProbeTabletNum       | 在探测模式下为缓存命中生成的平板电脑数。 |
| CacheProbeTime（缓存探测时间）            | 探测模式下花费的计算时间。          |

`CachePopulate`*`XXX`* 指标提供有关更新查询缓存的查询缓存未命中的统计信息。

`CachePassthrough`*`XXX`* 指标提供有关查询缓存未命中的统计信息，因为生成的每个 tablet 计算结果的大小很大，因此查询缓存未更新。

`CacheProbe`*`XXX`* 指标提供有关查询缓存命中的统计信息。

在多版本缓存机制中，`CachePopulate` 指标和 `CacheProbe` 指标可能包含相同的 tablet 统计信息，`CachePassthrough` 指标和 `CacheProbe` 指标也可能包含相同的 tablet 统计数据。例如，当 StarRocks 计算每个 tablet 的数据时，会命中该 tablet 的历史版本生成的计算结果。在这种情况下，StarRocks 会将历史版本产生的增量数据读取到最新版本的 tablet，并对其进行计算，并将增量数据与缓存数据进行合并。如果合并后产生的计算结果大小不超过 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 参数指定的阈值，则将 tablet 的统计信息收集到 `CachePopulate` 指标中。否则，tablet 的统计信息将收集到 `CachePassthrough` 指标中。

## RESTful API 操作

- `metrics |grep query_cache`

  该 API 操作用于查询与查询缓存相关的指标。

  ```shell
  curl -s  http://<be_host>:<be_http_port>/metrics |grep query_cache
  
  # TYPE starrocks_be_query_cache_capacity gauge
  starrocks_be_query_cache_capacity 536870912
  # 类型 starrocks_be_query_cache_hit_count gauge
  starrocks_be_query_cache_hit_count 5084393
  # 类型 starrocks_be_query_cache_hit_ratio gauge
  starrocks_be_query_cache_hit_ratio 0.984098
  # 类型 starrocks_be_query_cache_lookup_count gauge
  starrocks_be_query_cache_lookup_count 5166553
  # 类型 starrocks_be_query_cache_usage gauge
  starrocks_be_query_cache_usage 0
  # 类型 starrocks_be_query_cache_usage_ratio gauge
  starrocks_be_query_cache_usage_ratio 0.000000
  ```

- `api/query_cache/stat`

  该 API 操作用于查询查询缓存的使用情况。

  ```shell
  curl  http://<be_host>:<be_http_port>/api/query_cache/stat
  {
      "capacity": 536870912,
      "usage": 0,
      "usage_ratio": 0.0,
      "lookup_count": 5025124,
      "hit_count": 4943720,
      "hit_ratio": 0.983800598751394
  }
  ```

- `api/query_cache/invalidate_all`

  该 API 操作用于清除查询缓存。

  ```shell
  curl  -XPUT http://<be_host>:<be_http_port>/api/query_cache/invalidate_all
  
  {
      "status": "OK"
  }
  ```

上述 API 操作的参数如下：

- `be_host`：BE 所在节点的 IP 地址。
- `be_http_port`：BE 所在节点的 HTTP 端口号。

## 注意事项

- StarRocks 需要将首次发起的查询的计算结果填充到查询缓存中。因此，查询性能可能会略低于预期，并且查询延迟会增加。
- 如果配置较大的查询缓存大小，则可预配用于 BE 上的查询评估的内存量将减少。建议查询缓存大小不要超过为查询评估预配的内存容量的 1/6。
- 如果需要处理的平板电脑数小于 `pipeline_dop` 的值，则查询缓存不起作用。要使查询缓存正常工作，可以设置为 `pipeline_dop` 较小的值，例如 `1`。从 v3.0 开始，StarRocks 会根据查询并行度自适应调整该参数。

## 示例

### 数据集

1. 登录到您的 StarRocks 集群，转到目标数据库，并运行以下命令以创建名为 `t0` 的表：

   ```SQL
   CREATE TABLE t0
   (
         `ts` datetime NOT NULL COMMENT "",
         `k0` varchar(10) NOT NULL COMMENT "",
         `k1` char(6) NOT NULL COMMENT "",
         `v0` bigint(20) NOT NULL COMMENT "",
         `v1` decimal64(7, 2) NOT NULL COMMENT ""
   )
   ENGINE=OLAP 
   DUPLICATE KEY(`ts`, `k0`, `k1`)
   COMMENT "OLAP"
   PARTITION BY RANGE(`ts`)
   (
       START ("2022-01-01 00:00:00") END ("2022-02-01 00:00:00") EVERY (INTERVAL 1 DAY)
   )
   DISTRIBUTED BY HASH(`ts`, `k0`, `k1`)
   PROPERTIES
   (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "enable_persistent_index" = "false"
   );
   ```

2. 将以下数据记录插入 `t0`：

   ```SQL
   INSERT INTO t0
   VALUES
       ('2022-01-11 20:42:26', 'n4AGcEqYp', 'hhbawx', '799393174109549', '8029.42'),
       ('2022-01-27 18:17:59', 'i66lt', 'mtrtzf', '100400167', '10000.88'),
       ('2022-01-28 20:10:44', 'z6', 'oqkeun', '-58681382337', '59881.87'),
       ('2022-01-29 14:54:31', 'qQ', 'dzytua', '-19682006834', '43807.02'),
       ('2022-01-31 08:08:11', 'qQ', 'dzytua', '7970665929984223925', '-8947.74'),
       ('2022-01-15 00:40:58', '65', 'hhbawx', '4054945', '156.56'),
       ('2022-01-24 16:17:51', 'onqR3JsK1', 'udtmfp', '-12962', '-72127.53'),
       ('2022-01-01 22:36:24', 'n4AGcEqYp', 'fabnct', '-50999821', '17349.85'),
       ('2022-01-21 08:41:50', 'Nlpz1j3h', 'dzytua', '-60162', '287.06'),
       ('2022-01-30 23:44:55', '', 'slfght', '62891747919627339', '8014.98'),
       ('2022-01-18 19:14:28', 'z6', 'dzytua', '-1113001726', '73258.24'),
       ('2022-01-30 14:54:59', 'z6', 'udtmfp', '111175577438857975', '-15280.41'),
       ('2022-01-08 22:08:26', 'z6', 'ympyls', '3', '2.07'),
       ('2022-01-03 08:17:29', 'Nlpz1j3h', 'udtmfp', '-234492', '217.58'),
       ('2022-01-27 07:28:47', 'Pc', 'cawanm', '-1015', '-20631.50'),
       ('2022-01-17 14:07:47', 'Nlpz1j3h', 'lbsvqu', '2295574006197343179', '93768.75'),
       ('2022-01-31 14:00:12', 'onqR3JsK1', 'umlkpo', '-227', '-66199.05'),
       ('2022-01-05 20:31:26', '65', 'lbsvqu', '684307', '36412.49'),
       ('2022-01-06 00:51:34', 'z6', 'dzytua', '11700309310', '-26064.10'),
       ('2022-01-26 02:59:00', 'n4AGcEqYp', 'slfght', '-15320250288446', '-58003.69'),
       ('2022-01-05 03:26:26', 'z6', 'cawanm', '19841055192960542', '-5634.36'),
       ('2022-01-17 08:51:23', 'Pc', 'ghftus', '35476438804110', '13625.99'),
       ('2022-01-30 18:56:03', 'n4AGcEqYp', 'lbsvqu', '3303892099598', '8.37'),
       ('2022-01-22 14:17:18', 'i66lt', 'umlkpo', '-27653110', '-82306.25'),
       ('2022-01-02 10:25:01', 'qQ', 'ghftus', '-188567166', '71442.87'),
       ('2022-01-30 04:58:14', 'Pc', 'ympyls', '-9983', '-82071.59'),
       ('2022-01-05 00:16:56', '7Bh', 'hhbawx', '43712', '84762.97'),
       ('2022-01-25 03:25:53', '65', 'mtrtzf', '4604107', '-2434.69'),
       ('2022-01-27 21:09:10', '65', 'udtmfp', '476134823953365199', '38736.04'),
       ('2022-01-11 13:35:44', '65', 'qmwhvr', '1', '0.28'),
       ('2022-01-03 19:13:07', '', 'lbsvqu', '11', '-53084.04'),
       ('2022-01-20 02:27:25', 'i66lt', 'umlkpo', '3218824416', '-71393.20'),
       ('2022-01-04 04:52:36', '7Bh', 'ghftus', '-112543071', '-78377.93'),
       ('2022-01-27 18:27:06', 'Pc', 'umlkpo', '477', '-98060.13'),
       ('2022-01-04 19:40:36', '', 'udtmfp', '433677211', '-99829.94'),
       ('2022-01-20 23:19:58', 'Nlpz1j3h', 'udtmfp', '361394977', '-19284.18'),
       ('2022-01-05 02:17:56', 'Pc', 'oqkeun', '-552390906075744662', '-25267.92'),
       ('2022-01-02 16:14:07', '65', 'dzytua', '132', '2393.77'),
       ('2022-01-28 23:17:14', 'z6', 'umlkpo', '61', '-52028.57'),
       ('2022-01-12 08:05:44', 'qQ', 'hhbawx', '-9579605666539132', '-87801.81'),
       ('2022-01-31 19:48:22', 'z6', 'lbsvqu', '9883530877822', '34006.42'),
       ('2022-01-11 20:38:41', '', 'piszhr', '56108215256366', '-74059.80'),
       ('2022-01-01 04:15:17', '65', 'cawanm', '-440061829443010909', '88960.51'),
       ('2022-01-05 07:26:09', 'qQ', 'umlkpo', '-24889917494681901', '-23372.12'),
       ('2022-01-29 18:13:55', 'Nlpz1j3h', 'cawanm', '-233', '-24294.42'),
       ('2022-01-10 00:49:45', 'Nlpz1j3h', 'ympyls', '-2396341', '77723.88'),
       ('2022-01-29 08:02:58', 'n4AGcEqYp', 'slfght', '45212', '93099.78'),
       ('2022-01-28 08:59:21', 'onqR3JsK1', 'oqkeun', '76', '-78641.65'),
       ('2022-01-26 14:29:39', '7Bh', 'umlkpo', '176003552517', '-99999.96'),
       ('2022-01-03 18:53:37', '7Bh', 'piszhr', '3906151622605106', '55723.01'),
       ('2022-01-04 07:08:19', 'i66lt', 'ympyls', '-240097380835621', '-81800.87'),
       ('2022-01-28 14:54:17', 'Nlpz1j3h', 'slfght', '-69018069110121', '90533.64'),
       ('2022-01-22 07:48:53', 'Pc', 'ympyls', '22396835447981344', '-12583.39'),
       ('2022-01-22 07:39:29', 'Pc', 'uqkghp', '10551305', '52163.82'),
       ('2022-01-08 22:39:47', 'Nlpz1j3h', 'cawanm', '67905472699', '87831.30'),

       ('2022-01-05 14:53:34', '7Bh', 'dzytua', '-779598598706906834', '-38780.41'),
       ('2022-01-30 17:34:41', 'onqR3JsK1', 'oqkeun', '346687625005524', '-62475.31'),
       ('2022-01-29 12:14:06', '', 'qmwhvr', '3315', '22076.88'),
       ('2022-01-05 06:47:04', 'Nlpz1j3h', 'udtmfp', '-469', '42747.17'),
       ('2022-01-19 15:20:20', '7Bh', 'lbsvqu', '347317095885', '-76393.49'),
       ('2022-01-08 16:18:22', 'z6', 'fghmcd', '2', '90315.60'),
       ('2022-01-02 00:23:06', 'Pc', 'piszhr', '-3651517384168400', '58220.34'),
       ('2022-01-12 08:23:31', 'onqR3JsK1', 'udtmfp', '5636394870355729225', '33224.25'),
       ('2022-01-28 10:46:44', 'onqR3JsK1', 'oqkeun', '-28102078612755', '6469.53'),
       ('2022-01-23 23:16:11', 'onqR3JsK1', 'ghftus', '-707475035515433949', '63422.66'),
       ('2022-01-03 05:32:31', 'z6', 'hhbawx', '-45', '-49680.52'),
       ('2022-01-27 03:24:33', 'qQ', 'qmwhvr', '375943906057539870', '-66092.96'),
       ('2022-01-25 20:07:22', '7Bh', 'slfght', '1', '72440.21'),
       ('2022-01-04 16:07:24', 'qQ', 'uqkghp', '751213107482249', '16417.31'),
       ('2022-01-23 19:22:00', 'Pc', 'hhbawx', '-740731249600493', '88439.40'),
       ('2022-01-05 09:04:20', '7Bh', 'cawanm', '23602', '302.44');
   ```

### 查询示例

本节中查询缓存相关指标的统计信息仅为示例，仅供参考。

#### 阶段 1 的本地聚合支持查询缓存

这包括三种情况：

- 查询仅访问单个平板电脑。
- 查询访问来自表的多个分区的多个平板电脑，该表本身包括一个并置组，并且不需要对数据进行洗牌以进行聚合。
- 查询访问来自表的同一分区的多个平板电脑，并且不需要对数据进行洗牌以进行聚合。

查询示例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    k0,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    k0
```

下图显示了查询配置文件中与查询缓存相关的指标。

![查询缓存 - 阶段 1 - 指标](../assets/query_cache_stage1_agg_with_cache_en.png)

#### 阶段 1 的远程聚合不支持查询缓存

当在阶段 1 强制执行多个平板电脑上的聚合时，数据首先进行洗牌，然后进行聚合。

查询示例：

```SQL
SET new_planner_agg_stage = 1;

SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

#### 阶段 2 的本地聚合支持查询缓存

这包括三种情况：

- 查询阶段 2 的聚合编译以比较相同类型的数据。第一个聚合是本地聚合。第一次聚合完成后，将计算第一次聚合生成的结果，以执行第二次聚合，即全局聚合。
- 查询是 SELECT DISTINCT 查询。
- 查询包括以下 DISTINCT 聚合函数之一： `sum(distinct)`、 `count(distinct)`和 `avg(distinct)`。在大多数情况下，此类查询的聚合是在第 3 阶段或第 4 阶段执行的。但是，您可以运行 `set new_planner_agg_stage = 1` 以强制执行查询的阶段 2 的聚合。如果查询包含 `avg(distinct)` 并且您希望在阶段执行聚合，则还需要运行 `set cbo_cte_reuse = false` 以禁用 CTE 优化。

查询示例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

下图显示了查询配置文件中与查询缓存相关的指标。

![查询缓存 - 阶段 2 - 指标](../assets/query_cache_stage2_agg_with_cache_en.png)

#### 阶段 3 的本地聚合支持查询缓存

该查询是包含单个 DISTINCT 聚合函数的 GROUP BY 聚合查询。

支持的 DISTINCT 聚合函数为 `sum(distinct)`、 `count(distinct)`和 `avg(distinct)`。

> **注意**
>
> 如果查询包括 `avg(distinct)`，则还需要运行 `set cbo_cte_reuse = false` 以禁用 CTE 优化。

查询示例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd;
```

下图显示了查询配置文件中与查询缓存相关的指标。

![查询缓存 - 阶段 3 - 指标](../assets/query_cache_stage3_agg_with_cache_en.png)

#### 阶段 4 的本地聚合支持查询缓存

该查询是包含单个 DISTINCT 聚合函数的非 GROUP BY 聚合查询。此类查询包括删除重复数据的经典查询。

查询示例：

```SQL
SELECT
    count(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
```

下图显示了查询配置文件中与查询缓存相关的指标。

![查询缓存 - 阶段 4 - 指标](../assets/query_cache_stage4_agg_with_cache_en.png)

#### 两个查询的第一个聚合在语义上是等效的，缓存的结果将重用

以以下两个查询 Q1 和 Q2 为例。Q1 和 Q2 都包含多个聚合，但它们的第一个聚合在语义上是等效的。因此，Q1 和 Q2 在语义上被评估为等价，并且可以重用保存在查询缓存中的彼此计算结果。

- 查询 Q1

  ```SQL
  SELECT
      (
          ifnull(sum(murmur_hash3_32(hour)), 0) + ifnull(sum(murmur_hash3_32(k0)), 0) + ifnull(sum(murmur_hash3_32(__c_0)), 0)
        ) AS fingerprint
  FROM
      (
          SELECT
              date_trunc('hour', ts) AS hour,
              k0,
              sum(v1) AS __c_0
          FROM
              t0
          WHERE
              ts between '2022-01-03 00:00:00'
              and '2022-01-03 23:59:59'
          GROUP BY
              date_trunc('hour', ts),
              k0
      ) AS t;
  ```

- 查询 Q2

  ```SQL
  SELECT
      date_trunc('hour', ts) AS hour,
      k0,
      sum(v1) AS __c_0
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59'
  GROUP BY
      date_trunc('hour', ts),
      k0
  ```

下图显示了 Q1 的 `CachePopulate` 指标。

![查询缓存 - Q1 - 指标](../assets/query_cache_reuse_Q1_en.png)

下图显示了 Q2 的 `CacheProbe` 指标。

![查询缓存 - Q2 - 指标](../assets/query_cache_reuse_Q2_en.png)

#### 启用 CTE 优化的 DISTINCT 查询不支持查询缓存

在运行 `set cbo_cte_reuse = true` 以启用 CTE 优化后，特定查询包含 DISTINCT 聚合函数的计算结果将无法缓存。以下是几个例子：

- 查询包含单个 DISTINCT 聚合函数 `avg(distinct)`：

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![查询缓存 - CTE - 1](../assets/query_cache_distinct_with_cte_Q1_en.png)

- 查询包含多个引用同一列的 DISTINCT 聚合函数：

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0,
      sum(distinct v1) AS __c_1,
      count(distinct v1) AS __c_2
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![查询缓存 - CTE - 2](../assets/query_cache_distinct_with_cte_Q2_en.png)

- 查询包含多个 DISTINCT 聚合函数，每个函数引用不同的列：

  ```SQL
  SELECT
      sum(distinct v1) AS __c_1,
      count(distinct v0) AS __c_2
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![查询缓存 - CTE - 3](../assets/query_cache_distinct_with_cte_Q3_en.png)


## 最佳实践

在创建表时，请指定合理的分区描述和合理的分布方法，包括：

- 选择单个 DATE 类型的列作为分区列。如果表包含多个 DATE 类型的列，请选择值在增量摄入数据时会不断推移的列，该列用于定义查询的有趣时间范围。
- 选择适当的分区宽度。最近摄入的数据可能会修改表的最新分区。因此，涉及最新分区的缓存条目不稳定，容易失效。
- 在创建表时的分布描述中指定几十个存储桶编号。如果存储桶数量过少，则当 BE 需要处理的 tablet 数量小于 `pipeline_dop` 的值时，查询缓存无法生效。