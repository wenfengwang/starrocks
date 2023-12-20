---
displayed_sidebar: English
---

# 查询缓存

查询缓存是 StarRocks 的一个强大特性，它可以显著提升聚合查询的性能。通过在内存中存储本地聚合的中间结果，查询缓存可以避免对于与之前相同或相似的新查询进行不必要的磁盘访问和计算。有了查询缓存，StarRocks 能够为聚合查询提供快速且准确的结果，节约时间和资源，提高可扩展性。查询缓存特别适用于高并发场景，即许多用户对大型复杂数据集执行相似查询的情况。

该功能自 v2.5 版本起支持。

在 v2.5 版本中，查询缓存仅支持单个扁平表上的聚合查询。自 v3.0 版本起，查询缓存也支持在星型模式下连接的多个表上进行聚合查询。

## 应用场景

我们建议在以下场景中使用查询缓存：

- 您经常对单个扁平表或以星型模式连接的多个表执行聚合查询。
- 您的大多数聚合查询是非 GROUP BY 聚合查询和低基数 GROUP BY 聚合查询。
- 您的数据按时间分区以追加模式加载，并可根据访问频率划分为热数据和冷数据。

查询缓存支持满足以下条件的查询：

- 查询引擎为 Pipeline。要启用 Pipeline 引擎，请将会话变量 `enable_pipeline_engine` 设置为 `true`。

    > **注意**
    > 其他查询引擎不支持查询缓存。

- 查询针对原生 OLAP 表（自 v2.5 起）或云原生表（自 v3.0 起）。查询缓存不支持对外部表的查询。查询缓存还支持计划中需要访问同步物化视图的查询。但是，查询缓存不支持计划中需要访问异步物化视图的查询。

- 查询是对单个表或多个联接表的聚合查询。

  **注意**
  - 查询缓存支持 Broadcast Join 和 Bucket Shuffle Join。
  - 查询缓存支持包含 Join 运算符的两种树结构：Aggregation-Join 和 Join-Aggregation。在 Aggregation-Join 树结构中不支持 Shuffle Join，而在 Join-Aggregation 树结构中不支持 Hash Join。

- 查询不包含非确定性函数，如 `rand`、`random`、`uuid` 和 `sleep`。

查询缓存支持对使用以下任何分区策略的表进行查询：无分区、多列分区和单列分区。

## 功能边界

- 查询缓存基于 Pipeline 引擎的每个 Tablet 计算。每个 Tablet 计算意味着一个 Pipeline 驱动程序可以一次处理整个 Tablet，而不是处理 Tablet 的一部分或多个 Tablet 交错在一起。如果每个单独的 BE 需要为查询处理的 Tablet 数量大于或等于调用的 Pipeline 驱动程序的数量，则查询缓存有效。调用的 Pipeline 驱动程序数量代表实际的并行度（DOP）。如果 Tablet 数量小于 Pipeline 驱动程序的数量，每个 Pipeline 驱动程序只处理特定 Tablet 的一部分。在这种情况下，无法生成每个 Tablet 的计算结果，因此查询缓存不起作用。
- 在 StarRocks 中，聚合查询至少包含四个阶段。只有当 OlapScanNode 和 AggregateNode 从同一片段计算数据时，第一阶段的 AggregateNode 生成的每个 Tablet 计算结果才能被缓存。其他阶段的 AggregateNode 生成的每个 Tablet 计算结果不能被缓存。对于一些 DISTINCT 聚合查询，如果会话变量 `cbo_cte_reuse` 设置为 `true`，当产生数据的 OlapScanNode 和消费产生数据的第一阶段 AggregateNode 计算不同片段的数据并通过 ExchangeNode 连接时，查询缓存不起作用。以下两个示例展示了执行 CTE 优化导致查询缓存不起作用的场景：
  - 输出列通过聚合函数 `avg(distinct)` 计算得出。
  - 输出列通过多个 DISTINCT 聚合函数计算得出。
- 如果您的数据在聚合前被打乱，查询缓存无法加速对该数据的查询。
- 如果表的分组列或去重列是高基数列，对该表的聚合查询将生成大量结果。在这些情况下，查询将在运行时绕过查询缓存。
- 查询缓存占用 BE 提供的少量内存来保存计算结果。查询缓存的默认大小为 512 MB。因此，查询缓存不适合保存大型数据项。此外，启用查询缓存后，如果缓存命中率较低，查询性能会下降。因此，如果为某个 Tablet 生成的计算结果大小超过 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 参数指定的阈值，则查询缓存不再对该查询起作用，查询将切换到 Passthrough 模式。

## 工作原理

启用查询缓存后，每个 BE 将查询的本地聚合分为以下两个阶段：

1. 每个 Tablet 的聚合

   BE 单独处理每个 Tablet。当 BE 开始处理一个 Tablet 时，它首先探测查询缓存以查看该 Tablet 上聚合的中间结果是否在查询缓存中。如果是（缓存命中），BE 直接从查询缓存中获取中间结果。如果不是（缓存未命中），BE 访问磁盘上的数据并执行本地聚合以计算中间结果。当 BE 完成对一个 Tablet 的处理后，它会将该 Tablet 上聚合的中间结果填充到查询缓存中。

2. Tablet 间聚合

   BE 收集查询中涉及的所有 Tablet 的中间结果，并将它们合并为最终结果。

   ![查询缓存 - 工作原理 - 1](../assets/query_cache_principle-1.png)

当将来发出类似的查询时，它可以重用先前查询的缓存结果。例如，下图所示的查询涉及三个 Tablet（Tablet 0 到 2），第一个 Tablet（Tablet 0）的中间结果已在查询缓存中。对于此示例，BE 可以直接从查询缓存中获取 Tablet 0 的结果，而不是访问磁盘上的数据。如果查询缓存完全预热，它可以包含所有三个 Tablet 的中间结果，因此 BE 不需要访问磁盘上的任何数据。

![查询缓存 - 工作原理 - 2](../assets/query_cache_principle-2.png)

为了释放额外的内存，查询缓存采用基于最近最少使用（LRU）的淘汰策略来管理其中的缓存条目。根据这一淘汰策略，当查询缓存占用的内存量超过其预定义大小（`query_cache_capacity`）时，最近最少使用的缓存条目将被淘汰出查询缓存。

> **注意**
> 未来，StarRocks 也将支持基于生存时间（TTL）的淘汰策略，根据该策略可以将缓存条目从查询缓存中淘汰。

FE 确定每个查询是否需要使用查询缓存来加速，并规范化查询，消除对查询语义无影响的琐碎文字细节。

为了防止查询缓存的不良案例导致性能损失，BE 采用自适应策略在运行时绕过查询缓存。

## 启用查询缓存

本节介绍用于启用和配置查询缓存的参数和会话变量。

### FE 会话变量

|**变量**|**默认值**|**可动态配置**|**描述**|
|---|---|---|---|
|enable_query_cache|false|是|指定是否启用查询缓存。有效值：`true` 和 `false`。`true` 表示启用此功能，`false` 表示禁用此功能。当启用查询缓存时，它仅适用于满足本主题“应用场景”部分指定条件的查询。|
|query_cache_entry_max_bytes|4194304|是|指定触发 Passthrough 模式的阈值。有效值：`0` 到 `9223372036854775807`。当查询访问的特定 Tablet 的计算结果的字节数或行数超过 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 参数指定的阈值时，查询将切换到 Passthrough 模式。<br />如果 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 参数设置为 `0`，即使没有生成计算结果，也会使用 Passthrough 模式。|
|query_cache_entry_max_rows|409600|是|同上。|

### BE 参数

您需要在 BE 配置文件 **be.conf** 中配置以下参数。在重新配置此参数后，您必须重启 BE 以使新的参数设置生效。

|**参数**|**必需**|**描述**|
|---|---|---|
|query_cache_capacity|否|指定查询缓存的大小。单位：字节。默认大小为 512 MB。<br />每个 BE 在内存中都有自己的本地查询缓存，并且只填充和探测自己的查询缓存。<br />请注意，查询缓存大小不能小于 4 MB。如果 BE 的内存容量不足以配置您预期的查询缓存大小，您可以增加 BE 的内存容量。|

## 针对所有场景的最大缓存命中率设计

考虑三种情况，即使查询在字面上不完全相同，查询缓存仍然有效。这三种情况是：

- 语义等价的查询
- 具有重叠扫描分区的查询
- 针对仅有追加数据变更的数据的查询（无 UPDATE 或 DELETE 操作）

### 语义等价的查询

```
当两个查询在语义上相似时，并不意味着它们必须在字面上完全相同，而是指它们在执行计划中包含语义等效的片段。广义上讲，如果两个查询从相同的数据源查询数据、使用相同的计算方法，并且具有相同的执行计划，则它们被认为在语义上是等效的。StarRocks 应用以下规则来评估两个查询是否在语义上等效：

- 如果两个查询包含多个聚合操作，只要它们的第一个聚合操作在语义上等效，它们就被认为在语义上等效。例如，以下两个查询 Q1 和 Q2 都包含多个聚合操作，但它们的第一个聚合操作在语义上等效。因此，Q1 和 Q2 被认为在语义上等效。

-   Q1

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
                ts BETWEEN '2022-01-03 00:00:00'
                AND '2022-01-03 23:59:59'
            GROUP BY
                date_trunc('hour', ts),
                k0
        ) AS t;
    ```

-   Q2

    ```SQL
    SELECT
        date_trunc('hour', ts) AS hour,
        k0,
        sum(v1) AS __c_0
    FROM
        t0
    WHERE
        ts BETWEEN '2022-01-03 00:00:00'
        AND '2022-01-03 23:59:59'
    GROUP BY
        date_trunc('hour', ts),
        k0
    ```

- 如果两个查询都属于以下查询类型之一，它们可以被评估为语义等效。注意，包含 HAVING 子句的查询不能被评估为与不包含 HAVING 子句的查询语义等效。但是，包含 ORDER BY 或 LIMIT 子句不影响评估两个查询是否在语义上等效。

-   GROUP BY 聚合

    ```SQL
    SELECT <GroupByItems>, <AggFunctionItems> 
    FROM <Table> 
    WHERE <Predicates> [AND <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    [HAVING <HavingPredicate>] 
    ```

        > **注意**
        > 在上述示例中，HAVING 子句是可选的。

-   GROUP BY DISTINCT 聚合

    ```SQL
    SELECT DISTINCT <GroupByItems>, <Items> 
    FROM <Table> 
    WHERE <Predicates> [AND <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    HAVING <HavingPredicate>
    ```

        > **注意**
        > 在上述示例中，HAVING 子句是可选的。

-   非 GROUP BY 聚合

    ```SQL
    SELECT <AggFunctionItems> FROM <Table> 
    WHERE <Predicates> [AND <PartitionColumnRangePredicate>]
    ```

-   非 GROUP BY DISTINCT 聚合

    ```SQL
    SELECT DISTINCT <Items> FROM <Table> 
    WHERE <Predicates> [AND <PartitionColumnRangePredicate>]
    ```

- 如果任一查询包含 `PartitionColumnRangePredicate`，则在评估两个查询的语义等效性之前应移除 `PartitionColumnRangePredicate`。`PartitionColumnRangePredicate` 指定引用分区列的以下类型的谓词之一：

  - `col BETWEEN v1 AND v2`：分区列的值落在 [v1, v2] 范围内，其中 `v1` 和 `v2` 是常量表达式。
  - `v1 < col AND col < v2`：分区列的值落在 (v1, v2) 范围内，其中 `v1` 和 `v2` 是常量表达式。
  - `v1 < col AND col <= v2`：分区列的值落在 (v1, v2] 范围内，其中 `v1` 和 `v2` 是常量表达式。
  - `v1 <= col AND col < v2`：分区列的值落在 [v1, v2) 范围内，其中 `v1` 和 `v2` 是常量表达式。
  - `v1 <= col AND col <= v2`：分区列的值落在 [v1, v2] 范围内，其中 `v1` 和 `v2` 是常量表达式。

- 如果两个查询的 SELECT 子句的输出列在重新排列后相同，则这两个查询被评估为语义等效。

- 如果两个查询的 GROUP BY 子句的输出列在重新排列后相同，则这两个查询被评估为语义等效。

- 如果删除 `PartitionColumnRangePredicate` 后两个查询的 WHERE 子句的剩余谓词在语义上等效，则这两个查询被评估为语义等效。

- 如果两个查询的 HAVING 子句中的谓词在语义上等效，则这两个查询被评估为语义等效。

以下是表 `lineorder_flat` 的示例：

```SQL
CREATE TABLE `lineorder_flat`
(
    -- [字段定义省略，与原文相同]
)
ENGINE=OLAP 
DUPLICATE KEY(`lo_orderdate`, `lo_orderkey`)
COMMENT "olap"
PARTITION BY RANGE(`lo_orderdate`)
-- [分区定义省略，与原文相同]
DISTRIBUTED BY HASH(`lo_orderkey`)
PROPERTIES 
-- [属性定义省略，与原文相同]
);
```

以下两个查询 Q1 和 Q2 对表 `lineorder_flat` 在经过如下处理后在语义上是等效的：

1. 重新排列 SELECT 语句的输出列。
2. 重新排列 GROUP BY 子句的输出列。
3. 删除 ORDER BY 子句的输出列。
4. 重新排列 WHERE 子句中的谓词。
5. 添加 `PartitionColumnRangePredicate`。

- Q1

  ```SQL
  SELECT sum(lo_revenue), year(lo_orderdate) AS year, p_brand
  FROM lineorder_flat
  WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA'
  GROUP BY year, p_brand
  ORDER BY year, p_brand;
  ```

- Q2

  ```SQL
  SELECT year(lo_orderdate) AS year, p_brand, sum(lo_revenue)
  FROM lineorder_flat
  WHERE s_region = 'AMERICA' AND p_category = 'MFGR#12' AND 
     lo_orderdate >= '1993-01-01' AND lo_orderdate <= '1993-12-31'
  GROUP BY p_brand, year(lo_orderdate)
  ```

语义等效性是基于查询的物理计划来评估的。因此，查询中的字面差异不会影响语义等效性的评估。此外，查询中的常量表达式会被移除，`CAST` 表达式在查询优化期间也会被移除，因此这些表达式不会影响语义等效性的评估。最后，列和关系的别名也不会影响语义等效性的评估。

### 具有重叠扫描分区的查询

查询缓存支持基于谓词的查询分割。

根据谓词语义分割查询有助于实现部分计算结果的复用。当查询包含引用表的分区列的谓词，并且谓词指定了值范围时，StarRocks 可以根据表分区将范围分割成多个区间。每个单独的区间的计算结果可以被其他查询单独复用。

以下是表 `t0` 的示例：

```SQL
CREATE TABLE IF NOT EXISTS t0
(
    ts DATETIME NOT NULL,
    k0 VARCHAR(10) NOT NULL,
    k1 BIGINT NOT NULL,
    v1 DECIMAL64(7, 2) NOT NULL 
)
ENGINE=OLAP
DUPLICATE KEY(`ts`, `k0`, `k1`)
COMMENT "OLAP"
-- [后续定义省略，与原文相同]
```
PARTITION BY RANGE(ts)
(
  START ("2022-01-01 00:00:00") END ("2022-02-01 00:00:00") EVERY (INTERVAL 1 DAY) 
)
DISTRIBUTED BY HASH(`ts`, `k0`, `k1`)
PROPERTIES
(
    "replication_num" = "1", 
    "storage_format" = "DEFAULT"
);
```

表 `t0` 按天分区，列 `ts` 是表的分区列。在以下四个查询中，Q2、Q3 和 Q4 可以重用为 Q1 缓存的部分计算结果：

- Q1

  ```SQL
  SELECT date_trunc('day', ts) AS day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-14 23:59:59'
  GROUP BY day;
  ```

  Q1 的 '2022-01-02 12:30:00' 到 '2022-01-14 23:59:59' 之间的谓词 `ts` 指定的取值范围可以分为以下区间：

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
  SELECT date_trunc('day', ts) AS day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-02 12:30:00' AND ts < '2022-01-05 00:00:00'
  GROUP BY day;
  ```

  Q2 可以重用 Q1 的以下区间内的计算结果：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ```

- Q3

  ```SQL
  SELECT date_trunc('day', ts) AS day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-01 12:30:00' AND ts <= '2022-01-10 12:00:00'
  GROUP BY day;
  ```

  Q3 可以重用 Q1 的以下区间内的计算结果：

  ```SQL
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  8. [2022-01-09 00:00:00, 2022-01-10 00:00:00),
  ```

- Q4

  ```SQL
  SELECT date_trunc('day', ts) AS day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-02 23:59:59'
  GROUP BY day;
  ```

  Q4 可以重用 Q1 的以下区间内的计算结果：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  ```

对部分计算结果重用的支持取决于所使用的分区策略，如下表所述。

| 分区策略 | 支持部分计算结果复用 |
|---|---|
| 未分区 | 不支持 |
| 多列分区 | 不支持<br />**注意**<br />将来可能支持此功能。 |
| 单列分区 | 支持 |

### 对仅附加数据更改的数据进行查询

查询缓存支持多版本缓存。

随着数据加载，新版本的平板电脑就会生成。因此，从以前版本的平板电脑生成的缓存计算结果会变得陈旧并落后于最新的平板电脑版本。在这种情况下，多版本缓存机制尝试将查询缓存中保存的陈旧结果和磁盘上存储的平板电脑的增量版本合并为平板电脑的最终结果，以便新的查询可以携带最新的平板电脑版本。多版本缓存受到表类型、查询类型和数据更新类型的约束。

对多版本缓存的支持因表类型和查询类型而异，如下表所述。

| 表类型 | 查询类型 | 支持多版本缓存 |
|---|---|---|
| Duplicate Key 表 | 基表查询<br />同步物化视图查询 | 基表查询：在所有情况下都支持，增量平板电脑版本包含数据删除记录时除外。<br />同步物化视图查询：在所有情况下都支持，除了 GROUP BY、HAVING 或查询的 WHERE 子句引用聚合列。 |
| 聚合表 | 对基表的查询或对同步物化视图的查询 | 在除以下情况外的所有情况下均受支持：<br />基表的模式包含聚合函数 `replace`。<br />查询的 GROUP BY、HAVING 或 WHERE 子句引用聚合列。<br />增量平板电脑版本包含数据删除记录。 |
| Unique Key 表 | 不适用 | 不支持。但是，支持查询缓存。 |
| Primary Key 表 | 不适用 | 不支持。但是，支持查询缓存。 |

数据更新类型对多版本缓存的影响如下：

- 数据删除

  如果平板电脑的增量版本包含删除操作，则多版本缓存无法工作。

- 数据插入

  - 如果为某个平板电脑生成了空版本，则查询缓存中该平板电脑的现有数据仍然有效并且仍然可以检索。
  - 如果为某个平板电脑生成了非空版本，则该平板电脑在查询缓存中的现有数据仍然有效，但其版本落后于该平板电脑的最新版本。在这种情况下，StarRocks 会读取从现有数据的版本到最新版本的平板电脑生成的增量数据，将现有数据与增量数据合并，并将合并后的数据填充到查询缓存中。

- 架构更改和平板电脑截断

  如果表的模式发生更改或表的特定平板电脑被截断，则会为该表生成新的平板电脑。结果，查询缓存中该表的平板电脑的现有数据变得无效。

## 指标

查询缓存工作的查询配置文件包含 `CacheOperator` 统计信息。

在查询的源计划中，如果管道包含 `OlapScanOperator`，则 `OlapScanOperator` 和聚合运算符的名称均以 `ML_` 为前缀，以表示管道使用 `MultilaneOperator` 执行每片计算。`CacheOperator` 插入在 `ML_CONJUGATE_AGGREGATE` 之前，以处理控制查询缓存如何在 Passthrough、Populate 和 Probe 模式下运行的逻辑。查询的配置文件包含以下 `CacheOperator` 指标，可帮助您了解查询缓存的使用情况。

| 指标 | 描述 |
|---|---|
| CachePassthroughBytes | Passthrough 模式下生成的字节数。 |
| CachePassthroughChunkNum | Passthrough 模式下生成的 chunk 数量。 |
| CachePassthroughRowNum | Passthrough 模式下生成的行数。 |
| CachePassthroughTabletNum | Passthrough 模式下生成的平板电脑数量。 |
| CachePassthroughTime: | Passthrough 模式下所花费的计算时间。 |
| CachePopulateBytes | Populate 模式下生成的字节数。 |
| CachePopulateChunkNum | Populate 模式下生成的块数。 |
| CachePopulateRowNum | Populate 模式下生成的行数。 |
| CachePopulateTabletNum | Populate 模式下生成的平板电脑数量。 |
| CachePopulateTime | Populate 模式下所花费的计算时间。 |
| CacheProbeBytes | Probe 模式下为缓存命中生成的字节数。 |
| CacheProbeChunkNum | Probe 模式下为缓存命中生成的块数。 |
| CacheProbeRowNum | Probe 模式下为缓存命中生成的行数。 |
| CacheProbeTabletNum | Probe 模式下为缓存命中生成的平板电脑数量。 |
| CacheProbeTime | Probe 模式下所花费的计算时间。 |

`CachePopulate`*`XXX`* 指标提供有关查询缓存未命中的统计信息，该查询缓存已更新。

`CachePassthrough`*`XXX`* 指标提供有关查询缓存未命中的统计信息，这些未命中的查询缓存由于生成的每片计算结果的大小较大而未更新。

`CacheProbe`*`XXX`* 指标提供有关查询缓存命中的统计信息。

在多版本缓存机制中，`CachePopulate` 指标和 `CacheProbe` 指标可以包含相同的平板电脑统计信息，并且 `CachePassthrough` 指标和 `CacheProbe` 指标也可以包含相同的平板电脑统计信息。例如，当 StarRocks 计算每个平板电脑的数据时，都会命中该平板电脑历史版本上生成的计算结果。在这种情况下，StarRocks 会读取平板电脑历史版本到最新版本产生的增量数据，进行数据计算，并将增量数据与缓存数据合并。如果合并后生成的计算结果的大小未超过 `query_cache_entry_max_bytes` 或 `query_cache_entry_max_rows` 参数指定的阈值，则将平板电脑的统计信息收集到 `CachePopulate` 指标中。否则，平板电脑的统计信息将收集到 `CachePassthrough` 指标中。

## RESTful API 操作

- `metrics | grep query_cache`

  该 API 操作用于查询查询缓存相关的指标。

  ```shell
  curl -s  http://<be_host>:<be_http_port>/metrics | grep query_cache
  
  # TYPE starrocks_be_query_cache_capacity gauge
  starrocks_be_query_cache_capacity 536870912
  # TYPE starrocks_be_query_cache_hit_count gauge
  starrocks_be_query_cache_hit_count 5084393
  # TYPE starrocks_be_query_cache_hit_ratio gauge
  starrocks_be_query_cache_hit_ratio 0.984098
  # TYPE starrocks_be_query_cache_lookup_count gauge
  ```


```markdown
      sum(distinct v1) AS __c_1,
      count(distinct v0) AS __c_2
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 3](../assets/query_cache_distinct_with_cte_Q3_en.png)

## 最佳实践

创建表时，请指定合理的分区描述和分布方法，包括：

- 选择一个 DATE 类型的列作为分区列。如果表中包含多个 DATE 类型的列，请选择随着数据增量导入而逐渐增加的列，并且该列用于定义查询的感兴趣时间范围。
- 选择合适的分区宽度。最近导入的数据可能会修改表的最新分区。因此，涉及最新分区的缓存条目是不稳定的，容易被使无效。
- 在创建表的语句中为分布描述指定一个合适的桶数，通常为几十个。如果桶数过小，当 BE 需要处理的 tablet 数量小于 `pipeline_dop` 的值时，查询缓存将不会生效。