---
displayed_sidebar: "Chinese"
---

# 排序流式聚合

数据库系统中常见的聚合方法包括哈希聚合和排序聚合。

从v2.5开始，StarRocks支持**排序流式聚合**。

## 工作原理

聚合节点（AGG）主要负责处理GROUP BY和聚合函数。

排序流式聚合可以通过比较GROUP BY键的顺序来对数据进行分组，而无需创建哈希表。这有效地减少了聚合所占用的内存资源。对于聚合基数较高的查询，排序流式聚合提高了聚合性能并减少了内存使用。

您可以通过设置以下变量来启用排序流式聚合：

```SQL
set enable_sort_aggregate=true;
```

## 限制

- GROUP BY中的键必须具有顺序。例如，如果排序键为 `k1, k2, k3`，那么：
  - 允许 `GROUP BY k1` 和 `GROUP BY k1, k2`。
  - `GROUP BY k1, k3` 不符合排序键顺序。因此，排序流式聚合无法对这样的子句生效。
- 所选分区必须是单个分区（因为相同的键可能分布在不同分区的不同机器上）。
- GROUP BY键必须与创建表时指定的桶键具有相同的分布。例如，如果表有三列 `k1, k2, k3`，桶键可以是 `k1` 或 `k1, k2`。
  - 如果桶键为 `k1`，`GROUP BY` 键可以是 `k1`, `k1, k2`, 或 `k1, k2, k3`。
  - 如果桶键为 `k1, k2`，`GROUP BY` 键可以是 `k1, k2` 或 `k1, k2, k3`。
  - 如果查询计划不符合此要求，则即使启用了排序流式聚合特性，也无法生效。
- 排序流式聚合仅适用于第一阶段的聚合（即在 AGG 节点下只有一个 Scan 节点）。

## 示例

1. 创建表并插入数据。

    ```SQL
    CREATE TABLE `test_sorted_streaming_agg_basic`
    (
        `id_int` int(11) NOT NULL COMMENT "",
        `id_string` varchar(100) NOT NULL COMMENT ""
    ) 
    ENGINE=OLAP 
    DUPLICATE KEY(`id_int`)COMMENT "OLAP"
    DISTRIBUTED BY HASH(`id_int`)
    PROPERTIES
    ("replication_num" = "3"); 

    INSERT INTO test_sorted_streaming_agg_basic VALUES
    (1, 'v1'),
    (2, 'v2'),
    (3, 'v3'),
    (1, 'v4');
    ```

2. 启用排序流式聚合并使用 EXPLAIN 查询 SQL 语句的概要信息。

    ```SQL
    set enable_sort_aggregate = true;

    explain costs select id_int, max(id_string)
    from test_sorted_streaming_agg_basic
    group by id_int;
    ```

## 检查是否启用了排序流式聚合

查看 `EXPLAIN costs` 的结果。如果 AGG 节点中的 `sorted streaming` 字段为 `true`，则表示已启用此功能。

```Plain
|                                                                                                                                    |
|   1:AGGREGATE (update finalize)                                                                                                    |
|   |  aggregate: max[([2: id_string, VARCHAR, false]); args: VARCHAR; result: VARCHAR; args nullable: false; result nullable: true] |
|   |  group by: [1: id_int, INT, false]                                                                                             |
|   |  sorted streaming: true                                                                                                        |
|   |  cardinality: 1                                                                                                                |
|   |  column statistics:                                                                                                            |
|   |  * id_int-->[-Infinity, Infinity, 0.0, 1.0, 1.0] UNKNOWN                                                                       |
|   |  * max-->[-Infinity, Infinity, 0.0, 1.0, 1.0] UNKNOWN                                                                          |
|   |                                                                                                                                |
|   0:OlapScanNode                                                                                                                   |
|      table: test_sorted_streaming_agg_basic, rollup: test_sorted_streaming_agg_basic                                               |
|      preAggregation: on                                                                                                            |
|      partitionsRatio=1/1, tabletsRatio=10/10                                                                                       |
|      tabletList=30672,30674,30676,30678,30680,30682,30684,30686,30688,30690                                                        |
|      actualRows=0, avgRowSize=2.0                                                                                                  |
|      cardinality: 1                                                                                                                |
|      column statistics:                                                                                                            |
|      * id_int-->[-Infinity, Infinity, 0.0, 1.0, 1.0] UNKNOWN                                                                       |
|      * id_string-->[-Infinity, Infinity, 0.0, 1.0, 1.0] UNKNOWN                                                                    |
```