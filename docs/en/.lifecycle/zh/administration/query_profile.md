---
displayed_sidebar: English
---

# 分析查询配置文件

本节介绍如何查看查询配置文件。查询配置文件记录了查询涉及的所有工作节点的执行信息。查询配置文件可帮助您快速识别影响 StarRocks 集群查询性能的瓶颈。

## 启用查询配置文件

对于 v2.5 之前的 StarRocks 版本，您可以通过将变量 `is_report_success` 设置为 `true` 来启用查询配置文件：

```SQL
SET is_report_success = true;
```

对于 StarRocks v2.5 或更高版本，您可以通过将变量 `enable_profile` 设置为 `true` 来启用查询配置文件：

```SQL
SET enable_profile = true;
```

### 运行时配置文件

从 v3.1 版本开始，StarRocks 支持运行时配置文件功能，使您可以在查询完成之前访问查询配置文件。

要使用此功能，除了将 `enable_profile` 设置为 `true` 之外，还需要设置会话变量 `runtime_profile_report_interval`。`runtime_profile_report_interval`（单位：秒，默认值：`10`）控制配置文件报告间隔，默认设置为 10 秒，这意味着每当查询时间超过 10 秒时，运行时配置文件功能就会自动启用。

```SQL
SET runtime_profile_report_interval = 10;
```

运行时配置文件显示与任何查询配置文件相同的信息。您可以像分析常规查询配置文件一样对其进行分析，以获得有关正在运行的查询的性能的宝贵见解。

但是，运行时配置文件可能不完整，因为执行计划的某些运算符可能依赖于其他运算符。为了轻松区分正在运行的操作符和已完成的操作符，正在运行的操作符会标记为 `Status: Running`。

## 访问查询配置文件

> **注意**
> 如果您使用的是 StarRocks 企业版，可以使用 StarRocks Manager 来访问和可视化您的查询配置文件。

如果您使用的是 StarRocks 社区版，请按照以下步骤访问您的查询配置文件：

1. 在浏览器中输入 `http://<fe_ip>:<fe_http_port>`。
2. 在显示的页面上，单击顶部导航窗格上的 **queries**。
3. 在 **Finished Queries** 列表中，选择要检查的查询，然后单击 **Profile** 列中的链接。

![img](../assets/profile-1.png)

浏览器会将您重定向到包含相应查询配置文件的新页面。

![img](../assets/profile-2.png)

## 解释查询配置文件

### 查询配置文件的结构

以下是查询配置文件示例：

```SQL
Query:
  Summary:
  Planner:
  Execution Profile 7de16a85-761c-11ed-917d-00163e14d435:
    Fragment 0:
      Pipeline (id=2):
        EXCHANGE_SINK (plan_node_id=18):
        LOCAL_MERGE_SOURCE (plan_node_id=17):
      Pipeline (id=1):
        LOCAL_SORT_SINK (plan_node_id=17):
        AGGREGATE_BLOCKING_SOURCE (plan_node_id=16):
      Pipeline (id=0):
        AGGREGATE_BLOCKING_SINK (plan_node_id=16):
        EXCHANGE_SOURCE (plan_node_id=15):
    Fragment 1:
       ...
    Fragment 2:
       ...
```

查询配置文件由三个部分组成：

- Fragment：执行树。一个查询可以分为一个或多个片段。
- Pipeline：执行链。一个执行链没有分支。一个片段可以被分割成多个管道。
- Operator：管道由多个操作符组成。

![img](../assets/profile-3.png)

*一个片段由多个管道组成。*

### 关键指标

查询配置文件包含大量显示查询执行详细信息的指标。在大多数情况下，您只需要观察操作符的执行时间和它们处理的数据大小。找到瓶颈之后，就可以相应地解决它们。

#### 概要

|指标|描述|
|---|---|
|Total|查询消耗的总时间，包括规划、执行和分析所花费的时间。|
|QueryCpuCost|查询的总 CPU 时间成本。CPU 时间成本是针对并发进程进行汇总的。因此，该指标的值可能大于查询的实际执行时间。|
|QueryMemCost|查询的总内存成本。|

#### 操作符的通用指标

|指标|描述|
|---|---|
|OperatorTotalTime|操作符的总 CPU 时间成本。|
|PushRowNum|操作符推送的数据的总行数。|
|PullRowNum|操作符拉取的数据的总行数。|

#### 独特的指标

|指标|描述|
|---|---|
|IOTaskExecTime|所有 I/O 任务的总执行时间。|
|IOTaskWaitTime|所有 I/O 任务的总等待时间。|
|MorselsCount|I/O 任务总数。|

#### 扫描操作符

|指标|描述|
|---|---|
|Table|表名称。|
|ScanTime|总扫描时间。扫描在异步 I/O 线程池中执行。|
|TabletCount|Tablet 数量。|
|PushdownPredicates|被下推的谓词数量。|
|BytesRead|StarRocks 读取的数据大小。|
|CompressedBytesRead|StarRocks 读取的压缩数据大小。|
|IOTime|总 I/O 时间。|
|BitmapIndexFilterRows|通过 Bitmap 索引过滤掉的数据行数。|
|BloomFilterFilterRows|通过 Bloom Filter 过滤掉的数据行数。|
|SegmentRuntimeZoneMapFilterRows|通过运行时 Zone Map 过滤掉的数据行数。|
|SegmentZoneMapFilterRows|通过 Zone Map 过滤掉的数据行数。|
|ShortKeyFilterRows|通过 Short Key 过滤掉的数据行数。|
|ZoneMapIndexFilterRows|通过 Zone Map 索引过滤掉的数据行数。|

#### 交换操作符

|指标|描述|
|---|---|
|PartType|数据分布类型。有效值：`UNPARTITIONED`、`RANDOM`、`HASH_PARTITIONED` 和 `BUCKET_SHUFFLE_HASH_PARTITIONED`。|
|BytesSent|发送的数据大小。|
|OverallThroughput|总体吞吐量。|
|NetworkTime|数据包传输时间（不包括接收后处理的时间）。有关如何计算此指标以及可能遇到的异常情况的更多信息，请参阅下面的常见问题解答。|
|WaitTime|因发送方队列已满而需等待的时间。|

#### 聚合操作符

|指标|描述|
|---|---|
|GroupingKeys|分组键的名称（GROUP BY 列）。|
|AggregateFunctions|聚合函数。|
|AggComputeTime|聚合函数消耗的计算时间。|
|ExprComputeTime|表达式消耗的计算时间。|
|HashTableSize|哈希表的大小。|

#### 连接操作符

|指标|描述|
|---|---|
|JoinPredicates|JOIN 操作的谓词。|
|JoinType|JOIN 类型。|
|BuildBuckets|哈希表的桶数。|
|BuildHashTableTime|构建哈希表所用的时间。|
|ProbeConjunctEvaluateTime|Probe Conjunct 消耗的时间。|
|SearchHashTableTimer|搜索哈希表所用的时间。|

#### 窗口函数操作符

|指标|描述|
|---|---|
|ComputeTime|窗口函数消耗的计算时间。|
|PartitionKeys|分区键的名称（PARTITION BY 列）。|
|AggregateFunctions|聚合函数。|

#### 排序操作符

|指标|描述|
|---|---|
|SortKeys|排序键的名称（ORDER BY 列）。|
|SortType|结果排序类型：列出所有结果，或列出前 n 结果。|

#### 表函数操作符

|指标|描述|
|---|---|
|TableFunctionExecTime|表函数消耗的计算时间。|
|TableFunctionExecCount|表函数执行的次数。|

#### 投影操作符

|指标|描述|
|---|---|
|ExprComputeTime|表达式消耗的计算时间。|
|CommonSubExprComputeTime|公共子表达式消耗的计算时间。|

#### 本地交换操作符

|指标|描述|
|---|---|
|Type|本地交换类型。有效值：`Passthrough`、`Partition` 和 `Broadcast`。|
|ShuffleNum|随机播放的次数。仅当 `Type` 为 `Partition` 时，此指标才有效。|

#### Hive Connector

|指标|描述|
|---|---|
|ScanRanges|已扫描的 Tablet 数量。|
|ReaderInit|Reader 的初始化时间。|
|ColumnReadTime|Reader 读取和解析数据所消耗的时间。|
|ExprFilterTime|用于过滤表达式的时间。|
|RowsRead|读取的数据行数。|

#### 输入流

|指标|描述|
|---|---|
|AppIOBytesRead|应用层 I/O 任务读取的数据大小。|
|AppIOCounter|应用层 I/O 任务的数量。|
|AppIOTime|应用层 I/O 任务读取数据所消耗的总时间。|
|FSBytesRead|存储系统读取的数据大小。|
|FSIOCounter|存储层 I/O 任务的数量。|
|FSIOTime|存储层读取数据所消耗的总时间。|

### 操作符消耗的时间

- 对于 OlapScan 和 ConnectorScan 操作符，它们的时间消耗相当于 `OperatorTotalTime + ScanTime`。由于 Scan 操作符在异步 I/O 线程池中执行 I/O 操作，因此 ScanTime 表示异步 I/O 时间。
- Exchange 操作符的时间消耗相当于 `OperatorTotalTime + NetworkTime`。因为 Exchange 操作符在 bRPC 线程池中发送和接收数据包，所以 NetworkTime 代表网络传输所消耗的时间。
- 对于所有其他操作符，它们的时间成本是 `OperatorTotalTime`。

### 指标合并和 MIN/MAX

管道引擎是一种并行计算引擎。每个 Fragment 被分发到多台机器上并行处理，每台机器上的 Pipeline 作为多个并发实例并行执行。因此，在分析时，StarRocks 会合并相同的指标，并记录所有并发实例中每个指标的最小值和最大值。

针对不同类型的指标采用不同的合并策略：

- 时间指标是平均值。例如：
  - `OperatorTotalTime` 表示所有并发实例的平均时间成本。
  - `__MAX_OF_OperatorTotalTime` 是所有并发实例中的最大时间成本。
  - `__MIN_OF_OperatorTotalTime` 是所有并发实例中时间成本最小的。

```SQL
             - OperatorTotalTime: 2.192us
               - __MAX_OF_OperatorTotalTime: 2.502us
```
# 分析查询配置文件

本主题介绍如何检查查询配置文件。查询配置文件记录了查询涉及的所有工作节点的执行信息。查询配置文件可以帮助您快速识别影响 StarRocks 集群查询性能的瓶颈。

## 启用查询配置文件

对于 StarRocks v2.5 之前的版本，您可以通过将变量 `is_report_success` 设置为 `true` 来启用查询配置文件：

```SQL
SET is_report_success = true;
```

对于 StarRocks v2.5 或更高版本，您可以通过将变量 `enable_profile` 设置为 `true` 来启用查询配置文件：

```SQL
SET enable_profile = true;
```

## 访问查询配置文件

> **注意**
> 如果您使用的是 StarRocks 的企业版，您可以使用 StarRocks Manager 来访问和可视化查询配置文件。

如果您使用的是 StarRocks 的社区版，请按照以下步骤访问查询配置文件：

1. 在浏览器中输入 `http://<fe_ip>:<fe_http_port>`。
2. 在显示的页面上，点击顶部导航栏上的 **queries**。
3. 在 **Finished Queries** 列表中，选择要检查的查询，并点击 **Profile** 列中的链接。

![img](../assets/profile-1.png)

浏览器将重定向到一个包含相应查询配置文件的新页面。

![img](../assets/profile-2.png)

## 解释查询配置文件

### 查询配置文件的结构

以下是一个查询配置文件的示例：

```SQL
Query:
  Summary:
  Planner:
  Execution Profile 7de16a85-761c-11ed-917d-00163e14d435:
    Fragment 0:
      Pipeline (id=2):
        EXCHANGE_SINK (plan_node_id=18):
        LOCAL_MERGE_SOURCE (plan_node_id=17):
      Pipeline (id=1):
        LOCAL_SORT_SINK (plan_node_id=17):
        AGGREGATE_BLOCKING_SOURCE (plan_node_id=16):
      Pipeline (id=0):
        AGGREGATE_BLOCKING_SINK (plan_node_id=16):
        EXCHANGE_SOURCE (plan_node_id=15):
    Fragment 1:
       ...
    Fragment 2:
       ...
```

查询配置文件由三个部分组成：

- Fragment：执行树。一个查询可以分为一个或多个片段。
- Pipeline：执行链。执行链没有分支。一个片段可以分为多个管道。
- Operator：一个管道由多个操作符组成。

![img](../assets/profile-3.png)

*一个片段由多个管道组成。*

### 关键指标

查询配置文件包含大量指标，显示查询执行的详细信息。在大多数情况下，您只需要观察操作符的执行时间和它们处理的数据的大小。找到瓶颈后，您可以相应地解决它们。

#### 概要

|指标|描述|
|---|---|
|Total|查询消耗的总时间，包括计划、执行和配置文件的时间。|
|QueryCpuCost|查询的总 CPU 时间成本。CPU 时间成本是并发进程的聚合。因此，该指标的值可能大于查询的实际执行时间。|
|QueryMemCost|查询的总内存成本。|

#### 操作符的通用指标

|指标|描述|
|---|---|
|OperatorTotalTime|操作符的总 CPU 时间成本。|
|PushRowNum|操作符推送的数据的总行数。|
|PullRowNum|操作符拉取的数据的总行数。|

#### 特定指标

|指标|描述|
|---|---|
|IOTaskExecTime|所有 I/O 任务的总执行时间。|
|IOTaskWaitTime|所有 I/O 任务的总等待时间。|
|MorselsCount|I/O 任务的总数。|

#### 扫描操作符

|指标|描述|
|---|---|
|Table|表名。|
|ScanTime|扫描的总时间。扫描在异步 I/O 线程池中执行。|
|TabletCount|片的数量。|
|PushdownPredicates|下推的谓词数量。|
|BytesRead|StarRocks 读取的数据的大小。|
|CompressedBytesRead|StarRocks 读取的压缩数据的大小。|
|IOTime|总 I/O 时间。|
|BitmapIndexFilterRows|通过位图索引过滤的数据行数。|
|BloomFilterFilterRows|通过 Bloomfilter 过滤的数据行数。|
|SegmentRuntimeZoneMapFilterRows|通过运行时区域映射过滤的数据行数。|
|SegmentZoneMapFilterRows|通过区域映射过滤的数据行数。|
|ShortKeyFilterRows|通过短键过滤的数据行数。|
|ZoneMapIndexFilterRows|通过区域映射索引过滤的数据行数。|

#### 交换操作符

|指标|描述|
|---|---|
|PartType|数据分布类型。有效值：`UNPARTITIONED`、`RANDOM`、`HASH_PARTITIONED` 和 `BUCKET_SHUFFLE_HASH_PARTITIONED`。|
|BytesSent|发送的数据的大小。|
|OverallThroughput|总吞吐量。|
|NetworkTime|数据包传输时间（不包括后接收处理的时间）。有关如何计算此指标以及它如何引发异常的更多信息，请参见下面的常见问题解答。|
|WaitTime|由于发送方队列已满而等待的时间。|

#### 聚合操作符

|指标|描述|
|---|---|
|GroupingKeys|分组键（GROUP BY 列）的名称。|
|AggregateFunctions|聚合函数。|
|AggComputeTime|聚合函数的计算时间。|
|ExprComputeTime|表达式的计算时间。|
|HashTableSize|哈希表的大小。|

#### JOIN 操作符

|指标|描述|
|---|---|
|JoinPredicates|JOIN 操作的谓词。|
|JoinType|JOIN 类型。|
|BuildBuckets|哈希表的桶数。|
|BuildHashTableTime|构建哈希表所用的时间。|
|ProbeConjunctEvaluateTime|Probe Conjunct 所用的时间。|
|SearchHashTableTimer|搜索哈希表所用的时间。|

#### 窗口函数操作符

|指标|描述|
|---|---|
|ComputeTime|窗口函数的计算时间。|
|PartitionKeys|分区键（PARTITION BY 列）的名称。|
|AggregateFunctions|聚合函数。|

#### 排序操作符

|指标|描述|
|---|---|
|SortKeys|排序键（ORDER BY 列）的名称。|
|SortType|结果排序类型：列出所有结果或列出前 n 个结果。|

#### TableFunction 操作符

|指标|描述|
|---|---|
|TableFunctionExecTime|表函数的计算时间。|
|TableFunctionExecCount|表函数的执行次数。|

#### Project 操作符

|指标|描述|
|---|---|
|ExprComputeTime|表达式的计算时间。|
|CommonSubExprComputeTime|公共子表达式的计算时间。|

#### LocalExchange 操作符

|指标|描述|
|---|---|
|Type|本地交换类型。有效值：`Passthrough`、`Partition` 和 `Broadcast`。|
|ShuffleNum|洗牌的次数。此指标仅在 `Type` 为 `Partition` 时有效。|

#### Hive 连接器

|指标|描述|
|---|---|
|ScanRanges|扫描的片数。|
|ReaderInit|Reader 的初始化时间。|
|ColumnReadTime|Reader 读取和解析数据所用的时间。|
|ExprFilterTime|过滤表达式所用的时间。|
|RowsRead|读取的数据行数。|

#### 输入流

|指标|描述|
|---|---|
|AppIOBytesRead|应用层的 I/O 任务读取的数据大小。|
|AppIOCounter|应用层的 I/O 任务数。|
|AppIOTime|应用层的 I/O 任务读取数据所用的总时间。|
|FSBytesRead|存储系统读取的数据大小。|
|FSIOCounter|存储层的 I/O 任务数。|
|FSIOTime|存储层读取数据所用的总时间。|

### 操作符的时间消耗

- 对于 OlapScan 和 ConnectorScan 操作符，它们的时间消耗等于 `OperatorTotalTime + ScanTime`。因为扫描操作符在异步 I/O 线程池中执行 I/O 操作，所以 ScanTime 表示异步 I/O 时间。
- Exchange 操作符的时间消耗等于 `OperatorTotalTime + NetworkTime`。因为 Exchange 操作符在 bRPC 线程池中发送和接收数据包，所以 NetworkTime 表示网络传输的时间。
- 对于其他所有操作符，它们的时间成本为 `OperatorTotalTime`。

### 指标合并和 MIN/MAX

Pipeline 引擎是一个并行计算引擎。每个片段被分发到多台机器上进行并行处理，每台机器上的管道作为多个并发实例同时执行。因此，在配置文件分析过程中，StarRocks 合并相同的指标，并记录每个指标在所有并发实例中的最小值和最大值。

不同类型的指标采用不同的合并策略：

- 时间指标是平均值。例如：
  - `OperatorTotalTime` 表示所有并发实例的平均时间成本。
  - `__MAX_OF_OperatorTotalTime` 是所有并发实例中的最大时间成本。
  - `__MIN_OF_OperatorTotalTime` 是所有并发实例中的最小时间成本。

```SQL
             - OperatorTotalTime: 2.192us
               - __MAX_OF_OperatorTotalTime: 2.502us
               - __MIN_OF_OperatorTotalTime: 1.882us
```

- 非时间指标是总计。例如：
  - `PullChunkNum` 表示所有并发实例的总数。
  - `__MAX_OF_PullChunkNum` 是所有并发实例中的最大值。
  - `__MIN_OF_PullChunkNum` 是所有并发实例中的最小值。

  ```SQL
                 - PullChunkNum: 146.66K (146660)
                   - __MAX_OF_PullChunkNum: 24.45K (24450)
                   - __MIN_OF_PullChunkNum: 24.435K (24435)
  ```

- 一些特殊指标没有最小值和最大值，在所有并发实例中具有相同的值（例如 DegreeOfParallelism）。

#### MIN 和 MAX 之间的明显差异

通常，MIN 和 MAX 值之间的明显差异表明数据存在偏斜。这可能发生在聚合或 JOIN 操作期间。

```SQL
             - OperatorTotalTime: 2m48s
               - __MAX_OF_OperatorTotalTime: 10m30s
               - __MIN_OF_OperatorTotalTime: 279.170us
```

## 执行基于文本的配置文件分析

从 v3.1 开始，StarRocks 提供了一种更用户友好的基于文本的配置文件分析功能。此功能允许您高效地识别查询优化的瓶颈和机会。

### 分析现有查询

您可以通过其 `QueryID` 分析现有查询的配置文件，无论查询是正在运行还是已完成。

#### 列出配置文件

执行以下 SQL 语句列出现有的配置文件：

```sql
SHOW PROFILELIST;
```

示例：

```sql
MySQL > show profilelist;
+--------------------------------------+---------------------+-------+----------+--------------------------------------------------------------------------------------------------------------------------------------+
| QueryId                              | StartTime           | Time  | State    | Statement                                                                                                                            |
+--------------------------------------+---------------------+-------+----------+--------------------------------------------------------------------------------------------------------------------------------------+
| b8289ffc-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:27 | 86ms  | Finished | SELECT o_orderpriority, COUNT(*) AS order_count\nFROM orders\nWHERE o_orderdate >= DATE '1993-07-01'\n    AND o_orderdate < DAT ...  |
| b5be2fa8-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:23 | 67ms  | Finished | SELECT COUNT(*)\nFROM (\n    SELECT l_orderkey, SUM(l_extendedprice * (1 - l_discount)) AS revenue\n        , o_orderdate, o_sh ...  |
| b36ac9c6-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:19 | 320ms | Finished | SELECT COUNT(*)\nFROM (\n    SELECT s_acctbal, s_name, n_name, p_partkey, p_mfgr\n        , s_address, s_phone, s_comment\n    F ... |
| b037b245-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:14 | 175ms | Finished | SELECT l_returnflag, l_linestatus, SUM(l_quantity) AS sum_qty\n    , SUM(l_extendedprice) AS sum_base_price\n    , SUM(l_exten ...   |
| a9543cf4-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:02 | 40ms  | Finished | select count(*) from lineitem                                                                                                        |
+--------------------------------------+---------------------+-------+----------+--------------------------------------------------------------------------------------------------------------------------------------+
5 rows in set
Time: 0.006s


MySQL > show profilelist limit 1;
+--------------------------------------+---------------------+------+----------+-------------------------------------------------------------------------------------------------------------------------------------+
| QueryId                              | StartTime           | Time | State    | Statement                                                                                                                           |
+--------------------------------------+---------------------+------+----------+-------------------------------------------------------------------------------------------------------------------------------------+
| b8289ffc-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:27 | 86ms | Finished | SELECT o_orderpriority, COUNT(*) AS order_count\nFROM orders\nWHERE o_orderdate >= DATE '1993-07-01'\n    AND o_orderdate < DAT ... |
+--------------------------------------+---------------------+------+----------+-------------------------------------------------------------------------------------------------------------------------------------+
1 row in set
Time: 0.005s
```

此 SQL 语句允许您轻松获取与每个查询关联的 `QueryId`。`QueryId` 是进一步分析和调查的关键标识符。

#### 分析配置文件

一旦您获得了 `QueryId`，您可以使用 ANALYZE PROFILE 语句对特定查询进行更详细的分析。此 SQL 语句提供了更深入的见解，并有助于全面检查查询的性能特征和优化。

```sql
ANALYZE PROFILE FROM '<QueryId>' [, <plan_node_id>, ...]
```

默认情况下，分析输出仅显示每个操作符的最关键指标。但是，您可以指定一个或多个计划节点的 ID，以查看相应的指标。此功能可以更全面地检查查询的性能，并有助于有针对性的优化。

示例 1：在不指定计划节点 ID 的情况下分析配置文件：

![img](../assets/profile-16.png)

示例 2：分析配置文件并指定计划节点的 ID：

![img](../assets/profile-17.png)

ANALYZE PROFILE 语句提供了一种分析和理解运行时配置文件的增强方法。它区分不同状态的操作符，例如阻塞、运行和完成。此外，该语句还可以根据处理的行号全面了解整个进度以及各个操作符的进度，从而可以更深入地了解查询执行和性能。此功能进一步促进了 StarRocks 中查询的分析和优化。

示例 3：分析正在运行的查询的运行时配置文件：

![img](../assets/profile-20.png)

### 分析模拟查询

您还可以使用 EXPLAIN ANALYZE 语句模拟给定查询并分析其配置文件。

```sql
EXPLAIN ANALYZE <sql>
```

目前，EXPLAIN ANALYZE 支持两种类型的 SQL 语句：查询（SELECT）语句和 INSERT INTO 语句。您只能在 StarRocks 的本机 OLAP 表上模拟 INSERT INTO 语句并分析其配置文件。请注意，当您分析 INSERT INTO 语句的配置文件时，实际上不会插入任何数据。默认情况下，事务会中止，以确保在配置文件分析过程中不会对数据进行意外更改。

示例 1：分析给定查询的配置文件：

![img](../assets/profile-18.png)

示例 1：分析 INSERT INTO 操作的配置文件：

![img](../assets/profile-19.png)

## 可视化查询配置文件

如果您是 StarRocks Enterprise Edition 的用户，您可以通过 StarRocks Manager 可视化您的查询配置文件。

**Profile Overview** 页面显示一些摘要指标，包括总执行时间 `ExecutionWallTime`、I/O 指标、网络传输大小以及 CPU 和 I/O 时间的比例。

![img](../assets/profile-4.jpeg)

通过点击操作符（节点）的卡片，您可以在页面右侧查看其详细信息。有三个选项卡：

- **Node**：该操作符的核心指标。
- **Node Detail**：该操作符的所有指标。
- **Pipeline**：操作符所属管道的指标。您无需过多关注此选项卡，因为它仅与调度相关。

![img](../assets/profile-5.jpeg)

### 识别瓶颈

操作符所占时间比例越大，其卡片颜色越深。这有助于您轻松识别查询的瓶颈。

![img](../assets/profile-6.jpeg)

### 检查数据是否倾斜

点击占用时间比例较大的操作符的卡片，并检查其 `MaxTime` 和 `MinTime`。`MaxTime` 和 `MinTime` 之间的明显差异通常表明数据存在倾斜。

![img](../assets/profile-7.jpeg)

然后，点击 **Node Detail** 选项卡，并检查是否有任何指标显示异常。在此示例中，聚合操作符的指标 `PushRowNum` 显示数据倾斜。

![img](../assets/profile-8.jpeg)

### 检查分区或分桶策略是否生效

您可以通过查看相应查询计划使用 `EXPLAIN <sql_statement>` 来检查分区或分桶策略是否生效。

![img](../assets/profile-9.png)

### 检查是否使用了正确的物化视图

点击相应的 Scan 操作符，并检查 **Node Detail** 选项卡上的 `Rollup` 字段。

![img](../assets/profile-10.jpeg)

### 检查左表和右表的 JOIN 计划是否正确

通常，StarRocks 会选择较小的表作为 JOIN 的右表。如果查询配置文件显示其他情况，则可能存在异常。

![img](../assets/profile-11.jpeg)

### 检查 JOIN 的分布类型是否正确

Exchange 操作符根据数据分布类型分为三种类型：

- `UNPARTITIONED`：广播。数据被复制成多个副本并分发到多个 BE。
- `RANDOM`：轮询。
- `HASH_PARTITIONED` 和 `BUCKET_SHUFFLE_HASH_PARTITIONED`：洗牌。`HASH_PARTITIONED` 和 `BUCKET_SHUFFLE_HASH_PARTITIONED` 之间的区别在于用于计算哈希码的哈希函数。

对于 Inner Join，右表可以是 `HASH_PARTITIONED` 和 `BUCKET_SHUFFLE_HASH_PARTITIONED` 类型或 `UNPARTITIONED` 类型。通常，只有当右表的行数小于 100K 时才采用 `UNPARTITIONED` 类型。

在下面的示例中，Exchange 操作符的类型为 Broadcast，但操作符传输的数据大小远远超过阈值。

![img](../assets/profile-12.jpeg)

### 检查 JoinRuntimeFilter 是否生效

当 Join 的右子节点构建哈希表时，它会创建一个运行时过滤器。该运行时过滤器被发送到左子树，并且如果可能的话被下推到 Scan 操作符。您可以在 Scan 操作符的 **Node Detail** 选项卡上检查与 `JoinRuntimeFilter` 相关的指标。

![img](../assets/profile-13.jpeg)

## 常见问题解答

### 为什么 Exchange 操作符的时间成本异常？

![img](../assets/profile-14.jpeg)

Exchange 操作符的时间成本由 CPU 时间和网络时间两部分组成。网络时间依赖于系统时钟。网络时间的计算如下：

1. 发送方在调用 bRPC 接口发送数据包之前记录一个 `send_timestamp`。
2. 接收方在从 bRPC 接口接收数据包后记录一个 `receive_timestamp`（不包括接收后处理时间）。
3. 处理完成后，接收方发送响应并计算网络延迟。数据包传输延迟等于 `receive_timestamp` - `send_timestamp`。

如果机器之间的系统时钟不一致，则 Exchange 操作符的时间成本会出现异常。

### 为什么所有操作符的总时间成本明显小于查询执行时间？

可能的原因是，在高并发场景下，一些管道驱动程序虽然可调度，但由于排队而无法及时处理。等待时间不记录在操作符的指标中，而是记录在 `PendingTime`、`ScheduleTime` 和 `IOTaskWaitTime` 中。

示例：

从配置文件中，我们可以看到 `ExecutionWallTime` 约为 55 毫秒。然而，所有操作符的总时间成本小于 10 毫秒，明显小于 `ExecutionWallTime`。

![img](../assets/profile-15.jpeg)
### 为什么所有算子的总时间成本明显小于查询执行时间？

可能原因：在高并发场景下，尽管一些管道驱动器是可调度的，但由于排队，它们可能无法及时得到处理。等待时间并未记录在算子的指标中，而是记录在 `PendingTime`、`ScheduleTime` 和 `IOTaskWaitTime` 中。

示例：

从配置文件中，我们可以看到 `ExecutionWallTime` 大约为 55 毫秒。然而，所有算子的总时间成本不到 10 毫秒，这明显低于 `ExecutionWallTime`。
