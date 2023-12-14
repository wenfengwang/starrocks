---
displayed_sidebar: "Chinese"
---

# 查询缓存

查询缓存是StarRocks的一个强大功能，可以极大地提高聚合查询的性能。通过在内存中存储本地聚合的中间结果，查询缓存可以避免不必要的磁盘访问和计算，从而为与先前查询相同或类似的新查询提供快速准确的结果。借助其查询缓存，StarRocks可以为聚合查询提供快速准确的结果，节省时间和资源，并实现更好的可伸缩性。在高并发场景下，查询缓存尤其适用，因为在此类场景中，许多用户会对大型复杂数据集运行类似的查询。

此功能自v2.5版本开始支持。

在v2.5中，查询缓存仅支持单平面表的聚合查询。自v3.0以来，查询缓存还支持星型架构中多表连接的聚合查询。

## 应用场景

我们建议您在以下情况下使用查询缓存：

- 您经常在单个平面表上或在连接成星型架构的多个表上运行聚合查询。
- 大多数聚合查询是非GROUP BY聚合查询和低基数GROUP BY聚合查询。
- 您的数据以追加模式按时间分区加载，并且可以根据访问频率将其分类为热数据和冷数据。

查询缓存支持满足以下条件的查询：

- 查询引擎是Pipeline。要启用Pipeline引擎，将会话变量`enable_pipeline_engine`设置为`true`。

  > **注意**
  >
  > 其他查询引擎不支持查询缓存。

- 查询是在本地OLAP表（从v2.5开始）或云原生表（从v3.0开始）上的。查询缓存不支持对外部表的查询。查询缓存还支持其计划需要访问同步物化视图的查询。不过，查询缓存不支持其计划需要访问异步物化视图的查询。

- 查询是在单个表或多个连接的表上的聚合查询。

  **注意**
  >
  > - 查询缓存支持广播连接和桶洗牌连接。
  > - 查询缓存支持包含Join操作符的两种树结构：聚合-Join和Join-聚合。聚合-Join树结构中不支持洗牌连接，而Join-聚合树结构中不支持哈希连接。

- 查询不包括非确定性函数，如`rand`、`random`、`uuid`和`sleep`。

查询缓存支持对使用以下任一分区策略的表的查询：非分区、多列分区和单列分区。

## 功能边界

- 查询缓存基于Pipeline引擎的每个平板计算。每个平板计算意味着一个Pipeline驱动程序可以逐个处理整个平板，而不是处理特定平板的部分或多个平板交错在一起。如果在查询中，每个独立的BE需要处理的平板数量大于或等于调用以运行此查询的Pipeline驱动程序数量，查询缓存将起作用。调用的Pipeline驱动程序数量代表实际的并行度（DOP）。如果平板数量小于Pipeline驱动程序数量，每个Pipeline驱动程序只处理特定平板的一部分。在这种情况下，将无法产生平板计算结果，因此查询缓存将不起作用。
- 在StarRocks中，聚合查询至少包括四个阶段。第一阶段由AggregateNode生成的每个平板计算结果仅在OlapScanNode和AggregateNode从相同片段计算数据时才能被缓存。在其他阶段由AggregateNode生成的每个平板计算结果不能被缓存。对于某些DISTINCT聚合查询，如果会话变量`cbo_cte_reuse`设置为`true`，当从不同片段产生数据的OlapScanNode和消耗所产生数据的第1阶段AggregateNode由ExchangeNode桥接时，查询缓存将不起作用。以下两个示例展示了执行CTE优化的情景，因此查询缓存将不起作用：
  - 输出列通过使用聚合函数`avg(distinct)`计算。
  - 输出列通过多个DISTINCT聚合函数计算。
- 如果在聚合前对数据进行了洗牌，查询缓存无法加速对该数据的查询。
- 如果表的GROUP BY列或去重列是高基数列，将会生成该表的聚合查询的大结果。在这些情况下，运行时查询将绕过查询缓存。
- 查询缓存占用BE提供的少量内存来保存计算结果。查询缓存的大小默认为512 MB。因此，不适合查询缓存保存大型数据项。此外，启用查询缓存后，如果缓存命中率低，查询性能将下降。因此，如果为平板生成的计算结果的大小超过`query_cache_entry_max_bytes`或`query_cache_entry_max_rows`参数指定的阈值，则查询缓存将不再对查询起作用，并且查询将切换到旁路模式。

## 工作原理

当启用查询缓存时，每个BE将查询的本地聚合拆分为以下两个阶段：

1. 平板聚合

   BE逐个处理每个平板。当BE开始处理某个平板时，首先会探测查询缓存，查看该平板上聚合的中间结果是否在查询缓存中。如果是（缓存命中），BE直接从查询缓存中获取中间结果。如果否（缓存未命中），BE将访问磁盘上的数据，并执行本地聚合以计算中间结果。当BE完成处理某个平板时，将查询缓存填充为该平板上聚合的中间结果。

2. 平板间聚合

   BE从参与查询的所有平板中收集中间结果，并将它们合并为最终结果。

   ![查询缓存 - 工作原理 - 1](../assets/query_cache_principle-1.png)

当将来发出类似的查询时，可以重复使用先前查询的缓存结果。例如，下图所示的查询涉及三个平板（平板0到2），并且第一个平板（平板0）的中间结果已经在查询缓存中。在这种情况下，BE可以直接从查询缓存中获取平板0的结果，而无需访问磁盘上的数据。如果查询缓存已完全预热，它可以包含所有三个平板的中间结果，因此BE无需访问任何磁盘数据。

![查询缓存 - 工作原理 - 2](../assets/query_cache_principle-2.png)

为释放额外内存，查询缓存采用基于最近最少使用（LRU）的驱逐策略来管理其中的缓存条目。根据此驱逐策略，当查询缓存占用的内存量超过其预定义大小（`query_cache_capacity`）时，最近最少使用的缓存条目将从查询缓存中被驱逐出去。

> **注意**
>
> 将来，StarRocks还将支持基于Time to Live（TTL）的驱逐策略，根据此策略，缓存条目可以从查询缓存中被驱逐出去。

FE通过确定查询是否需要被查询缓存加速，并对查询进行规范化以消除对查询语义无影响的琐碎文字细节，防止查询缓存的不良情况导致的性能损耗。

为防止查询缓存的不良情况导致的性能损耗，BE采用自适应策略在运行时绕过查询缓存。

## 启用查询缓存

本节介绍了用于启用和配置查询缓存的参数和会话变量。

### FE会话变量

| **变量**                   | **默认值** | **可动态配置** | **描述**                                  |
| -------------------------- | ---------- | -------------- | ----------------------------------------- |
| enable_query_cache          | false      | 是             | 指定是否启用查询缓存。有效值：`true`和`false`。`true`表示启用此功能，`false`表示禁用此功能。启用查询缓存时，它仅对符合本主题"[应用场景](../using_starrocks/query_cache.md#application-scenarios)"中指定条件的查询工作。 |
| query_cache_entry_max_bytes | 4194304    | 是             | 指定触发旁路模式的阈值。有效值：`0`至`9223372036854775807`。当查询访问特定平板计算结果的字节数或行数超过`query_cache_entry_max_bytes`或`query_cache_entry_max_rows`参数指定的阈值时，查询将切换到旁路模式。<br />如果将`query_cache_entry_max_bytes`或`query_cache_entry_max_rows`参数设置为`0`，则即使未从涉及的平板产生计算结果，也将使用旁路模式。 |
| query_cache_entry_max_rows  | 409600     | 是             | 同上。                                    |

### BE参数

您需要在BE配置文件**be.conf**中配置以下参数。在为BE重新配置此参数后，必须重新启动BE才能使新的参数设置生效。

| **参数**        | **必需** | **描述**                                  |
| --------------- | -------- | ----------------------------------------- |
| query_cache_capacity | No | 指定查询缓存的大小。单元：字节。默认大小为512 MB。<br/> 每个BE都有自己的本地查询缓存，它只会填充和检索自己的查询缓存。<br/>请注意，查询缓存的大小不能小于4 MB。如果BE的内存容量不足以满足您期望的查询缓存大小，可以增加BE的内存容量。|

## 针对所有场景的最大缓存命中率设计

即使查询在字面上不完全相同，也需要考虑查询缓存在以下三种情况下仍然有效。这三种情况是：

- 语义等价的查询
- 具有重叠扫描分区的查询
- 针对仅追加数据更改的数据进行查询（不包括UPDATE或DELETE操作）

### 语义等价的查询

当两个查询相似时，并不意味着它们必须在字面上等价，而是指它们在执行计划中包含语义上等价的片段，它们被视为语义上等价并且可以重用彼此的计算结果。在广义上，如果两个查询从相同的资源查询数据，使用相同的计算方法并且具有相同的执行计划，则它们被视为语义上等价。StarRocks应用以下规则来评估两个查询是否语义上等价：

- 如果两个查询包含多个聚合函数，只要它们的第一个聚合函数在语义上等价，它们就被评估为语义上等价。例如，以下两个查询Q1和Q2都包含多个聚合函数，但它们的第一个聚合函数在语义上等价。因此，Q1和Q2被评估为语义上等价。

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

- 如果两个查询都属于以下查询类型之一，则它们可以被评估为语义上等价。请注意，包括HAVING子句的查询无法被评估为语义上等价于不包括HAVING子句的查询。但是，包含ORDER BY或LIMIT子句不影响两个查询是否语义上等价的评估。

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
    > 在上述示例中，HAVING子句是可选的。

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
    > 在上述示例中，HAVING子句是可选的。

  - 非GROUP BY 聚合

    ```SQL
    SELECT <AggFunctionItems> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

  - 非GROUP BY DISTINCT 聚合

    ```SQL
    SELECT DISTINCT <Items> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

- 如果任一查询包括`PartitionColumnRangePredicate`，在评估两个查询是否语义上等价之前，将删除`PartitionColumnRangePredicate`。`PartitionColumnRangePredicate`指定引用分区列的以下类型谓词之一：

  - `col between v1 and v2`：分区列的值在[v1, v2]范围内，其中`v1`和`v2`是常量表达式。
  - `v1 < col and col < v2`：分区列的值在(v1, v2)范围内，其中`v1`和`v2`是常量表达式。
  - `v1 < col and col <= v2`：分区列的值在(v1, v2]范围内，其中`v1`和`v2`是常量表达式。
  - `v1 <= col and col < v2`：分区列的值在[v1, v2)范围内，其中`v1`和`v2`是常量表达式。
  - `v1 <= col and col <= v2`：分区列的值在[v1, v2]范围内，其中`v1`和`v2`是常量表达式。

- 如果两个查询的SELECT子句的输出列在重新排列后相同，则这两个查询被评估为语义上等价。

- 如果两个查询的GROUP BY子句的输出列在重新排列后相同，则这两个查询被评估为语义上等价。

- 如果两个查询的WHERE子句的剩余谓词在移除`PartitionColumnRangePredicate`后在语义上等价，这两个查询被评估为语义上等价。

- 如果两个查询的HAVING子句中的谓词在语义上等价，则这两个查询被评估为语义上等价。

使用以下表`lineorder_flat`作为示例：

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
分区 p5 的值为[('1996-01-01'), ('1997-01-01')),
分区 p6 的
值为[('1997-01-01'), ('1998-01-01')),
分区 p7 的值为[('1998-01-01'), ('1999-01-01'))
根据`lo_orderkey` 哈希分布
其属性为
(
    "replication_num" = "1",
    "colocate_with" = "groupxx1",
    "storage_format" = "DEFAULT",
    "enable_persistent_index" = "false",
    "compression" = "LZ4"
);

对表`lineorder_flat`的下述两个查询 Q1 和 Q2 在经过如下处理后语义上是等价的：

1. 重新排列 SELECT 语句的输出列。
2. 重新排列 GROUP BY 子句的输出列。
3. 移除 ORDER BY 子句的输出列。
4. 重新排列 WHERE 子句的谓词。
5. 增加 `PartitionColumnRangePredicate`。

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

语义等价性是基于查询的物理计划进行评估的。因此，查询中的文字差异不会影响语义等价性的评估。此外，查询中的常量表达式在优化查询时被移除，`cast` 表达式在查询优化期间也被移除。因此，这些表达式不会影响语义等价性的评估。第三，列和关系的别名对语义等价性的评估也没有影响。

### 具有重叠扫描分区的查询

查询缓存支持基于谓词语义的查询拆分。

基于谓词语义的查询拆分有助于实现部分计算结果的重用。当查询包含引用表的分区列的谓词，并且该谓词指定了数值范围时，StarRocks 可以根据表的分区对范围进行多个间隔的拆分。每个单独间隔的计算结果可以被其他查询分别重用。

以表 `t0` 为例：

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
```

表 `t0` 按日进行分区，列 `ts` 是表的分区列。在以下四个查询中，Q2、Q3 和 Q4 可以重用为 Q1 缓存的部分计算结果：

- Q1

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-14 23:59:59'
  GROUP BY day;
  ```

  Q1 的谓词 `ts between '2022-01-02 12:30:00' 和 '2022-01-14 23:59:59'` 指定的数值范围可以被拆分为以下间隔：

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

  Q2 可以重用 Q1 中以下部分间隔的计算结果：

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

  Q3 可以重用 Q1 中以下部分间隔的计算结果：

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

  Q4 可以重用 Q1 中以下部分间隔的计算结果：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  ```

支持部分计算结果重用的程度取决于使用的分区策略，如下表所述。

| **分区策略**           | **支持重用部分计算结果**                  |
| --------------------- | --------------------------------------- |
| 未分区的表                | 不支持                                   |
| 多列分区                  | 不支持<br />**注意**<br />此功能可能在未来得到支持。 |
| 单列分区                  | 支持                                    |

### 针对具有追加变更的数据的查询

查询缓存支持多版本缓存。

随着数据加载进行，会生成新版本的数据表。因此，由上一个数据表版本生成的缓存计算结果将变得过期，并且落后于最新的数据表版本。在这种情况下，多版本缓存机制尝试将保存在查询缓存中的过期结果和存储在磁盘上的增量数据表版本合并为数据表的最终结果，以便新的查询可以携带最新的数据表版本。多版本缓存受到表类型、查询类型和数据更新类型的约束。

多版本缓存的支持程度取决于表类型和查询类型，如下表所示。

| **表类型**        | **查询类型**                    | **支持多版本缓存**                                                  |
| ---------------- | ------------------------------ | ----------------------------------------------------------------- |
| 重复键表              | <ul><li>基表上的查询</li><li>同步物化视图上的查询</li></ul> | <ul><li>基表上的查询：在增量数据表版本包含数据删除记录时除外的所有情况下均会得到支持。</li><li>同步物化视图上的查询：在查询的 GROUP BY、HAVING 或 WHERE 子句引用聚合列时除外的所有情况下均会得到支持。</li></ul> |
| 聚合表          | 基表上的查询或同步物化视图上的查询                           | 在以下所有情况下均得到支持：基表的架构包含聚合函数 `replace`。查询的 GROUP BY、HAVING 或 WHERE 子句引用聚合列。增量数据表版本包含数据删除记录。 |
| Unique Key table    | N/A                                                          | Not supported. However, the query cache is supported.        |
| Primary Key table   | N/A                                                          | Not supported. However, the query cache is supported.        |

对多版本缓存的数据更新类型的影响如下：

- 数据删除

  如果增量版本的表格包含删除操作，则多版本缓存无法工作。

- 数据插入

  - 如果为某个表生成了空版本，则该表在查询缓存中的现有数据仍然有效，可以继续检索。
  - 如果为某个表生成了非空版本，则该表在查询缓存中的现有数据仍然有效，但其版本滞后于该表的最新版本。在这种情况下，StarRocks会读取从现有数据版本生成的增量数据到该表的最新版本，将现有数据与增量数据合并，并将合并后的数据填充到查询缓存中。

- 模式更改和表截断

  如果更改了表的模式或表的特定部分被截断，则会为该表生成新的表格。因此，表在查询缓存中的现有数据将变为无效。

## 指标

查询缓存起作用的查询的概要包含`CacheOperator`统计信息。

在查询的源计划中，如果管道包含`OlapScanOperator`，那么`OlapScanOperator`和聚合算子的名称将以`ML_`为前缀，表示该管道使用`MultilaneOperator`执行每个表格的计算。 `CacheOperator`会被插入到`ML_CONJUGATE_AGGREGATE`之前，用于处理控制查询缓存在穿透、填充和探测模式下运行逻辑。查询的概要包含以下`CacheOperator`指标，帮助您了解查询缓存的使用情况。

| **指标**                   | **描述**                       |
| ------------------------- | ------------------------------------------------------------ |
| CachePassthroughBytes     | 穿透模式下生成的字节数。           |
| CachePassthroughChunkNum  | 穿透模式下生成的块数。                 |
| CachePassthroughRowNum    | 穿透模式下生成的行数。             |
| CachePassthroughTabletNum | 穿透模式下生成的表格数。            |
| CachePassthroughTime:     | 穿透模式下的计算时间。           |
| CachePopulateBytes        | 填充模式下生成的字节数。               |
| CachePopulateChunkNum     | 填充模式下生成的块数。         |
| CachePopulateRowNum       | 填充模式下生成的行数。         |
| CachePopulateTabletNum    | 填充模式下生成的表格数。        |
| CachePopulateTime         | 填充模式下的计算时间。           |
| CacheProbeBytes           | 探测模式下为缓存命中生成的字节数。  |
| CacheProbeChunkNum        | 探测模式下为缓存命中生成的块数。 |
| CacheProbeRowNum          | 探测模式下为缓存命中生成的行数。   |
| CacheProbeTabletNum       | 探测模式下为缓存命中生成的表格数。   |
| CacheProbeTime            | 探测模式下的计算时间。           |

`CachePopulate`*`XXX`* 指标提供有关查询缓存未命中的统计信息，因此需要更新查询缓存。

`CachePassthrough`*`XXX`* 指标提供有关查询缓存未命中的统计信息，因为生成的表格计算结果较大，所以无需更新查询缓存。

`CacheProbe`*`XXX`* 指标提供有关查询缓存命中的统计信息。

在多版本缓存机制中，`CachePopulate`指标和`CacheProbe`指标可能包含相同的表格统计信息，`CachePassthrough`指标和`CacheProbe`指标也可能包含相同的表格统计信息。例如，当StarRocks计算每个表格的数据时，它命中了在该表格的历史版本上生成的计算结果。在这种情况下，StarRocks会读取生成的增量数据，从历史版本到该表格的最新版本，计算数据，并将增量数据与缓存数据合并。如果合并后的计算结果的大小未超过`query_cache_entry_max_bytes`或`query_cache_entry_max_rows`参数指定的阈值，则该表格的统计信息将被收集到`CachePopulate`指标中。否则，该表格的统计信息将被收集到`CachePassthrough`指标中。

## RESTful API 操作

- `metrics |grep query_cache`

  该 API 操作用于查询与查询缓存相关的指标。

  ```shell
  curl -s  http://<be_host>:<be_http_port>/metrics |grep query_cache
  
  # TYPE starrocks_be_query_cache_capacity gauge
  starrocks_be_query_cache_capacity 536870912
  # TYPE starrocks_be_query_cache_hit_count gauge
  starrocks_be_query_cache_hit_count 5084393
  # TYPE starrocks_be_query_cache_hit_ratio gauge
  starrocks_be_query_cache_hit_ratio 0.984098
  # TYPE starrocks_be_query_cache_lookup_count gauge
  starrocks_be_query_cache_lookup_count 5166553
  # TYPE starrocks_be_query_cache_usage gauge
  starrocks_be_query_cache_usage 0
  # TYPE starrocks_be_query_cache_usage_ratio gauge
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

  该 API 操作用于清空查询缓存。

  ```shell
  curl  -XPUT http://<be_host>:<be_http_port>/api/query_cache/invalidate_all
  
  {
      "status": "OK"
  }
  ```

前述 API 操作中的参数说明如下：

- `be_host`：BE 所在节点的 IP 地址。
- `be_http_port`：BE 所在节点的 HTTP 端口号。

## 注意事项

- StarRocks 需要使用首次启动的查询的计算结果填充查询缓存。因此，查询性能可能比预期稍微低一些，查询延迟会增加。
- 如果配置了较大的查询缓存大小，则可以分配给 BE 上的查询评估的内存量会减少。建议查询缓存大小不要超过为查询评估分配的内存容量的六分之一。
- 如果需要处理的表格数量小于`pipeline_dop`的值，则查询缓存无法工作。为使查询缓存工作，您可以将`pipeline_dop`设置为较小的值，例如`1`。从 v3.0 版开始，StarRocks会根据查询的并行性自适应调整此参数。

## 示例

### 数据集

1. 登录到 StarRocks 集群，进入目标数据库，运行以下命令创建名为`t0`的表格：

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

2. 向`t0`表格中插入以下数据记录：

   ```SQL
   INSERT INTO t0
   VALUES
       ('2022-01-11 20:42:26', 'n4AGcEqYp', 'hhbawx', '799393174109549', '8029.42'),
       ('2022-01-27 18:17:59', 'i66lt', 'mtrtzf', '100400167', '10000.88'),
       ('2022-01-28 20:10:44', 'z6', 'oqkeun', '-58681382337', '59881.87'),
       ('2022-01-29 14:54:31', 'qQ', 'dzytua', '-19682006834', '43807.02'),
       ```
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

本节中关于查询缓存相关指标的统计信息仅供参考。

#### 查询缓存在阶段 1 对本地聚合起作用

这包括三种情况：

- 查询仅访问单个表格片段。
- 查询访问了表中包含共位组的多个分区的多个表格片段，且数据在进行聚合时无需进行洗牌。
- 查询访问了表中同一分区的多个表格片段，且数据在进行聚合时无需进行洗牌。

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

以下图展示了查询概要中与查询缓存相关的指标。

![查询缓存 - 阶段 1 - 指标](../assets/query_cache_stage1_agg_with_cache_cn.png)

#### 查询缓存在阶段 1 对远程聚合不起作用

当在阶段 1 强制对多个表格片段进行聚合时，首先需要进行洗牌，然后再进行聚合。

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

#### 查询缓存在阶段 2 对本地聚合起作用

这包括三种情况：

- 查询的阶段 2 聚合被编译为比较相同类型的数据。第一次聚合为本地聚合。完成第一次聚合后，从第一次聚合生成的结果进行第二次聚合，即全局聚合。
- 查询是 SELECT DISTINCT 查询。
- 查询包含以下 DISTINCT 聚合函数之一：`sum(distinct)`、`count(distinct)` 和 `avg(distinct)`。对大多数情况下，此类查询的聚合将在阶段 3 或 4 进行。但是您可以运行 `set new_planner_agg_stage = 1` 强制该查询在阶段 2 进行聚合。如果查询包含 `avg(distinct)`，并且您想要在阶段进行聚合，则还需要运行 `set cbo_cte_reuse = false` 来禁用 CTE 优化。

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

以下图展示了查询概要中与查询缓存相关的指标。

![查询缓存 - 阶段 2 - 指标](../assets/query_cache_stage2_agg_with_cache_cn.png)

#### 查询缓存在阶段 3 对本地聚合起作用

查询为包含单一 DISTINCT 聚合函数的 GROUP BY 聚合查询。

支持的 DISTINCT 聚合函数有 `sum(distinct)`、`count(distinct)` 和 `avg(distinct)`。

> **注意**
>
> 如果查询包含 `avg(distinct)`，还需要运行 `set cbo_cte_reuse = false` 来禁用 CTE 优化。

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

以下图展示了查询概要中与查询缓存相关的指标。

![查询缓存 - 阶段 3 - 指标](../assets/query_cache_stage3_agg_with_cache_cn.png)

#### 查询缓存在阶段 4 对本地聚合起作用

查询为不包含 GROUP BY 的 DISTINCT 聚合查询。此类查询包括经典的去重数据查询。

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

以下图展示了查询概要中与查询缓存相关的指标。

![查询缓存 - 阶段 4 - 指标](../assets/query_cache_stage4_agg_with_cache_cn.png)

#### 重用具有语义等效的首次聚合结果的两个查询的缓存结果

以查询 Q1 和 Q2 为例。Q1 和 Q2 均包含多个聚合，但它们的首次聚合在语义上等效。因此，Q1 和 Q2 被视为语义等效，并可以重用存储在查询缓存中的彼此的计算结果。

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

以下图展示了 Q1 的 `CachePopulate` 指标。

![查询缓存 - Q1 - 指标](../assets/query_cache_reuse_Q1_cn.png)

以下图展示了 Q2 的 `CacheProbe` 指标。

![查询缓存 - Q2 - 指标](../assets/query_cache_reuse_Q2_cn.png)

#### 具有启用 CTE 优化的 DISTINCT 查询不使用查询缓存

在运行`set cbo_cte_reuse = true`以启用CTE优化后，包括DISTINCT聚合函数的特定查询的计算结果将不能被缓存。以下是一些示例：

- 查询包含一个单个的DISTINCT聚合函数`avg(distinct)`：

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 1](../assets/query_cache_distinct_with_cte_Q1_en.png)

- 查询包含引用相同列的多个DISTINCT聚合函数：

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

![Query Cache - CTE - 2](../assets/query_cache_distinct_with_cte_Q2_en.png)

- 查询包含每个引用不同列的多个DISTINCT聚合函数：

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

![Query Cache - CTE - 3](../assets/query_cache_distinct_with_cte_Q3_en.png)

## 最佳实践

创建表时，指定合理的分区描述和合理的分发方式，包括：

- 选择单个DATE类型列作为分区列。如果表包含多个DATE类型列，请选择值随数据增量摄入而向前滚动，并用于定义查询的感兴趣时间范围的列。
- 选择合适的分区宽度。最近摄入的数据可能修改表的最新分区。因此，涉及最新分区的缓存条目是不稳定的，易于失效。
- 在表创建语句的分布描述中指定一个数十的桶数。如果桶数过小，当需要由BE处理的tablet数量少于`pipeline_dop`的值时，查询缓存将无法生效。