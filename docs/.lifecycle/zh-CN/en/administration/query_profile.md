---
displayed_sidebar: "Chinese"
---

# 分析查询配置文件

本主题描述如何检查查询配置文件。查询配置文件记录了查询中涉及的所有worker节点的执行信息。查询配置文件可帮助您快速识别影响StarRocks集群查询性能的瓶颈。

## 启用查询配置文件

对于早于v2.5的StarRocks版本，您可以通过将变量 `is_report_success` 设置为`true`来启用查询配置文件：

```SQL
SET is_report_success = true;
```

对于v2.5或更高版本的StarRocks，您可以通过将变量`enable_profile`设置为`true`来启用查询配置文件：

```SQL
SET enable_profile = true;
```

### 运行时配置文件

从v3.1开始，StarRocks支持运行时配置文件功能，使您能够在查询完成之前就访问查询配置文件。

要使用此功能，除了设置`enable_profile`为`true`之外，还需要设置会话变量`runtime_profile_report_interval`。`runtime_profile_report_interval`（单位：秒，默认值：`10`）管理配置文件报告间隔，默认设置为10秒，这意味着每当查询时间超过10秒时，运行时配置文件功能会自动启用。

```SQL
SET runtime_profile_report_interval = 10;
```

运行时配置文件显示与任何查询配置文件相同的信息。您可以像处理常规查询配置文件一样分析它，以获取有价值的运行查询性能见解。

但是，运行时配置文件可能是不完整的，因为执行计划的一些运算符可能依赖于其他运算符。为了方便区分正在运行的运算符和已完成的运算符，正在运行的运算符用`状态：运行`标记。

## 访问查询配置文件

> **注意**
>
> 如果您正在使用StarRocks的企业版，则可以使用StarRocks Manager来访问和可视化您的查询配置文件。

如果您正在使用StarRocks的社区版，请按照以下步骤访问您的查询配置文件：

1. 在浏览器中输入 `http://<fe_ip>:<fe_http_port>`。
2. 在显示的页面上，点击顶部导航窗格上的**查询**。
3. 在**已完成的查询**列表中，选择要检查的查询，然后单击**Profile**列中的链接。

![img](../assets/profile-1.png)

浏览器将您重定向到包含相应查询配置文件的新页面。

![img](../assets/profile-2.png)

## 解释查询配置文件

### 查询配置文件的结构

以下是一个示例查询配置文件：

```SQL
Query:
  概要:
  计划器:
  执行配置文件 7de16a85-761c-11ed-917d-00163e14d435:
    片段 0:
      流水线（id=2）:
        EXCHANGE_SINK（plan_node_id=18）:
        LOCAL_MERGE_SOURCE（plan_node_id=17）:
      流水线（id=1）:
        LOCAL_SORT_SINK（plan_node_id=17）:
        AGGREGATE_BLOCKING_SOURCE（plan_node_id=16）:
      流水线（id=0）:
        AGGREGATE_BLOCKING_SINK（plan_node_id=16）:
        EXCHANGE_SOURCE（plan_node_id=15）:
    片段 1:
       ...
    片段 2:
       ...
```

查询配置文件包括三个部分：

- 片段：执行树状结构。一个查询可以被划分为一个或多个片段。
- 流水线：执行链条。执行链条没有分支。一个片段可以被分成几个流水线。
- 运算符：一个流水线由多个运算符组成。

![img](../assets/profile-3.png)

*一个片段由多个流水线组成*

### 关键指标

查询配置文件涵盖了大量指标，显示了查询执行的细节。在大多数情况下，您只需要观察运算符的执行时间和它们处理的数据大小。找到瓶颈后，您可以相应地解决问题。

#### 概要

| 指标           | 描述                                     |
| -------------- | ---------------------------------------- |
| 总计           | 查询消耗的总时间，包括计划、执行和配置时间。 |
| QueryCpuCost   | 查询的总CPU时间成本。CPU时间成本会对并发进程进行汇总。因此，此指标的值可能大于查询的实际执行时间。 |
| QueryMemCost   | 查询的总内存成本。                       |

#### 运算符的通用指标

| 指标              | 描述                               |
| ----------------- | ---------------------------------- |
| OperatorTotalTime | 运算符的总CPU时间成本。            |
| PushRowNum        | 运算符推送的数据的总行数。         |
| PullRowNum        | 运算符拉取的数据的总行数。         |

#### 特定指标

| 指标            | 描述                     |
| -------------   | ----------------------- |
| IOTaskExecTime  | 所有I/O任务的总执行时间。 |
| IOTaskWaitTime  | 所有I/O任务的总等待时间。 |
| MorselsCount    | I/O任务的总数。          |

#### 扫描运算符

| 指标                         | 描述                                        |
| ---------------------------- | --------------------------------------------- |
| Table                        | 表名。                                      |
| ScanTime                     | 扫描的总时间。扫描是在异步I/O线程池中进行的。  |
| TabletCount                  | Tablet的数量。                               |
| PushdownPredicates           | 被推送下去的谓词的数量。                    |
| BytesRead                    | StarRocks读取的数据大小。                    |
| CompressedBytesRead          | StarRocks读取的压缩数据大小。                |
| IOTime                       | 总I/O时间。                                  |
| BitmapIndexFilterRows        | Bitmap索引过滤的数据行数。                   |
| BloomFilterFilterRows        | Bloomfilter过滤的数据行数。                  |
| SegmentRuntimeZoneMapFilterRows | 通过运行时Zone Map过滤的数据行数。        |
| SegmentZoneMapFilterRows        | 通过Zone Map过滤的数据行数。              |
| ShortKeyFilterRows              | 通过Short Key过滤的数据行数。             |
| ZoneMapIndexFilterRows          | 通过Zone Map索引过滤的数据行数。          |

#### 交换运算符

| 指标           | 描述                                     |
| -------------- | ---------------------------------------- |
| PartType       | 数据分发类型。有效值：`UNPARTITIONED`、`RANDOM`、`HASH_PARTITIONED`和`BUCKET_SHUFFLE_HASH_PARTITIONED`。 |
| BytesSent      | 发送的数据大小。                           |
| OverallThroughput | 总吞吐量。                            |
| NetworkTime    | 数据包传输时间（不包括接收后处理的时间）。有关此指标计算方式以及异常情况的详细信息，请参阅下面的常见问题解答。 |
| WaitTime       | 由于发送端队列已满而等待的时间。           |

#### 聚合运算符

| 指标                 | 描述                               |
| -------------------- | ---------------------------------- |
| GroupingKeys          | 分组键的名称（GROUP BY列）。        |
| AggregateFunctions   | 聚合函数。                          |
| AggComputeTime       | 聚合函数的计算时间。               |
| ExprComputeTime      | 表达式的计算时间。                 |
| HashTableSize        | 哈希表的大小。                      |

#### 连接运算符

| 指标                     | 描述                     |
| ----------------------- | ------------------------ |
| JoinPredicates         | 连接操作的谓词。          |
| JoinType               | 连接类型。                |
| BuildBuckets           | 哈希表的桶数量。          |
| BuildHashTableTime     | 构建哈希表的时间。        |
| ProbeConjunctEvaluateTime | 检查的时间。              |
| SearchHashTableTimer   | 搜索哈希表的时间。        |

#### 窗口函数运算符

| 指标           | 描述                               |
| -------------- | ---------------------------------- |
| ComputeTime    | 窗口函数的计算时间。               |
| PartitionKeys  | 分区键的名称（PARTITION BY列）。    |
| AggregateFunctions | 聚合函数。                        |

#### 排序运算符

| 指标    | 描述                                                  |
| ------- | ----------------------------------------------------- |
| SortKeys | 排序键的名称（ORDER BY列）。                            |
| SortType | 结果排序类型：列出所有结果，或列出前n个结果。          |

#### TableFunction运算符

| 指标                      | 描述                                         |
| ------------------------  | -------------------------------------------- |
| TableFunctionExecTime     | 表函数的计算时间。                           |
| TableFunctionExecCount    | 执行表函数的次数。                           |

#### 项目运算符

| 指标                  | 描述                                         |
| ----------------------- | ------------------------------------------- |
| ExprComputeTime         | 表达式的计算时间。                           |
| CommonSubExprComputeTime | 公共子表达式的计算时间。                     |

#### 本地交换运算符

| 指标     | 描述                                        |
| -------- | ------------------------------------------ |
| 类型     | 本地交换类型。有效值：`传递`、`分区`和`广播`。 |
| ShuffleNum | 分派数量。此指标仅在`类型`为`分区`时有效。   |

#### Hive连接器

| 指标             | 描述                             |
| ------------------ | -------------------------------- |
| ScanRanges        | 扫描的片段数。                    |
| ReaderInit        | 读取器的初始化时间。               |
| ColumnReadTime              | Time consumed by Reader to read and parse data.        |
| ExprFilterTime              | Time used to filter expressions.                       |
| RowsRead                    | Number of data rows that are read.                     |

#### 输入流

| 指标                         | 描述                               |
| ------------------------- | ------------------------------ |
| AppIOBytesRead            | I/O任务从应用程序层读取的数据大小。            |
| AppIOCounter              | 应用程序层的I/O任务数量。                     |
| AppIOTime                 | 应用程序层从读取数据开始花费的总时间。          |
| FSBytesRead               | 存储系统读取的数据大小。                       |
| FSIOCounter               | 存储层的I/O任务数量。                         |
| FSIOTime                  | 存储层读取数据花费的总时间。                    |

### 运算符的时间消耗

- 对于OlapScan和ConnectorScan运算符，它们的时间消耗等同于`OperatorTotalTime + ScanTime`。因为Scan运算符在异步I/O线程池中执行I/O操作，ScanTime代表异步I/O时间。
- Exchange运算符的时间消耗等同于`Operator总时间 + NetworkTime`。因为Exchange运算符在bRPC线程池中发送和接收数据包，NetworkTime代表网络传输消耗的时间。
- 对于其他所有运算符，它们的时间成本是`OperatorTotalTime`。

### 指标合并和MIN/MAX

Pipeline引擎是一个并行计算引擎。每个片段分布到多台机器进行并行处理，并且每台机器上的管道都以多个并发实例的形式并行执行。因此，在分析过程中，StarRocks合并相同的指标，并记录所有并发实例中每个指标的最小值和最大值。

对不同类型的指标采用不同的合并策略：

- 时间指标是平均值。例如：
  - `OperatorTotalTime`表示所有并发实例的平均时间成本。
  - `__MAX_OF_OperatorTotalTime`是所有并发实例中的最大时间成本。
  - `__MIN_OF_OperatorTotalTime`是所有并发实例中的最小时间成本。

```SQL
             - OperatorTotalTime: 2.192us
               - __MAX_OF_OperatorTotalTime: 2.502us
               - __MIN_OF_OperatorTotalTime: 1.882us
```

- 非时间指标是总和值。例如：
  - `PullChunkNum`表示所有并发实例的总数。
  - `__MAX_OF_PullChunkNum`是所有并发实例中的最大值。
  - `__MIN_OF_PullChunkNum`是所有并发实例中的最小值。

  ```SQL
                 - PullChunkNum: 146.66K (146660)
                   - __MAX_OF_PullChunkNum: 24.45K (24450)
                   - __MIN_OF_PullChunkNum: 24.435K (24435)
  ```

- 一些特殊的指标，它们没有最小值和最大值，在所有并发实例中具有相同的值（例如`DegreeOfParallelism`）。

#### MIN和MAX之间的显著差异

通常，MIN和MAX值之间的显著差异表示数据存在倾斜的可能。这可能发生在聚合或JOIN操作期间。

```SQL
             - OperatorTotalTime: 2m48s
               - __MAX_OF_OperatorTotalTime: 10m30s
               - __MIN_OF_OperatorTotalTime: 279.170us
```

## 执行基于文本的性能分析

从v3.1开始，StarRocks提供了更加用户友好的基于文本的性能分析功能。此功能允许您高效地识别瓶颈并找到查询优化的机会。

### 分析现有查询

您可以通过其`QueryID`分析现有查询的性能 profile，无论该查询是正在运行还是已经完成。

#### 列出profile

执行以下SQL语句以列出现有的profile：

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
| b5be2fa8-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:23 | 67ms  | Finished | SELECT COUNT(*)\nFROM (\n    SELECT l_orderkey, SUM(l_ex...

<此处内容被压缩，原文超出长度>

- finished | select count(*) from lineitem                                                                                                        |
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

该SQL语句允许您轻松获取与每个查询关联的`QueryId`。 `QueryId`作为进一步profile分析和调查的关键标识符。

#### 分析profile

一旦您获得了`QueryId`，您可以使用ANALYZE PROFILE语句对特定查询执行更详细的分析。此SQL语句提供了更深入的见解，并便于全面地检查查询的性能特征和优化。

```sql

ANALYZE PROFILE FROM '<QueryId>' [, <plan_node_id>, ...]

```

默认情况下，分析输出仅呈现每个运算符的最关键指标。但是，您可以指定一个或多个计划节点的ID来查看相应的指标。这个功能允许更全面地检查查询的性能，并便于有针对性地优化。

示例1：分析profile而不指定计划节点ID：

![img](../assets/profile-16.png)

示例2：分析profile并指定计划节点ID：

![img](../assets/profile-17.png)

ANALYZE PROFILE语句提供了一种增强的分析和了解运行时profile的方法。它区分具有不同状态的运算符，如blocked，running和finished。此外，该语句还提供了关于整个进度以及根据处理的行数对各个运算符的进度的全面视图，使您能够更深入地理解查询执行和性能。该功能进一步便于在StarRocks中进行查询的profile分析和优化。

示例3：分析正在运行查询的运行时profile：

![img](../assets/profile-20.png)

### 分析模拟查询

您还可以模拟给定查询并使用EXPLAIN ANALYZE语句分析其profile。

```sql

EXPLAIN ANALYZE <sql>

```

目前，EXPLAIN ANALYZE支持两种类型的SQL语句：查询（SELECT）语句和INSERT INTO语句。您只能模拟INSERT INTO语句并分析其profile，而且profile分析期间不会实际插入数据。默认情况下，事务会被中止，以确保在profile分析过程中不会对数据进行意外更改。

示例1：分析给定查询的profile：

如果你是StarRocks Enterprise Edition的用户，你可以通过StarRocks Manager可视化你的查询配置文件。

**性能概览**页面显示一些摘要指标，包括总执行时间`ExecutionWallTime`，I/O指标，网络传输大小和CPU和I/O时间的比例。

![img](../assets/profile-4.jpeg)

通过点击运算符（节点）的卡片，你可以在页面右侧窗格中查看其详细信息。有三个标签：

- **节点**：这个运算符的核心指标。
- **节点详细**：这个运算符的所有指标。
- **Pipeline**：运算符所属管道的指标。你不需要过多关注这个标签，因为这只与调度有关。

![img](../assets/profile-5.jpeg)

### 识别瓶颈

运算符所占用时间比例越大，其卡片颜色越深。这有助于您轻松识别查询的瓶颈。

![img](../assets/profile-6.jpeg)

### 检查数据是否倾斜

点击占用大部分时间的运算符卡片，检查其`MaxTime`和`MinTime`。`MaxTime`和`MinTime`之间明显的差异通常表示数据倾斜。

然后，点击**节点详细**标签，并检查是否有任何指标显示异常。在这个示例中，聚合运算符的指标`PushRowNum`显示数据倾斜。

![img](../assets/profile-7.jpeg)

### 检查分区或桶化策略是否生效

您可以通过使用`EXPLAIN <sql_statement>`查看相应的查询计划，以检查分区或桶化策略是否生效。

![img](../assets/profile-9.png)

### 检查是否使用了正确的物化视图

点击相应的扫描运算符，并在**节点详细**标签上检查`Rollup`字段。

![img](../assets/profile-10.jpeg)

### 检查左右表的JOIN计划是否合适

通常，StarRocks选择较小的表作为Join的右表。如果查询配置文件显示不同，则会发生异常。

![img](../assets/profile-11.jpeg)

### 检查JOIN的分布类型是否正确

Exchange运算符根据数据分布类型分为三种类型：

- `UNPARTITIONED`：广播。数据被复制并分发给多个BE。

- `RANDOM`：轮询。
- `HASH_PARTITIONED`和`BUCKET_SHUFFLE_HASH_PARTITIONED`：洗牌。`HASH_PARTITIONED`和`BUCKET_SHUFFLE_HASH_PARTITIONED`之间的区别在于用于计算哈希码的哈希函数。


对于内连接，右表可以是`HASH_PARTITIONED`和`BUCKET_SHUFFLE_HASH_PARTITIONED`类型或`UNPARTITIONED`类型。通常情况下，只有右表中的行数少于100K行时，才采用`UNPARTITIONED`类型。

在以下示例中，Exchange运算符的类型是广播，但由该运算符传输的数据量远远超过了阈值。

![img](../assets/profile-12.jpeg)

### 检查JoinRuntimeFilter是否生效

当Join的右子节点正在构建哈希表时，它会创建一个运行时过滤器。这个运行时过滤器被发送到左子节点树，并且如果可能的话，被推送到扫描运算符。您可以在扫描运算符的**节点详细**标签上检查与`JoinRuntimeFilter`相关的指标。

![img](../assets/profile-13.jpeg)

## 常见问题

### 为什么Exchange运算符的时间成本异常？

![img](../assets/profile-14.jpeg)

Exchange运算符的时间成本由两部分组成：CPU时间和网络时间。网络时间依赖于系统时钟。网络时间的计算如下：

1. 发送方在调用bRPC接口发送数据包之前记录`send_timestamp`。
2. 接收方在从bRPC接口接收数据包后（不包括接收后处理时间）记录`receive_timestamp`。
3. 处理完成后，接收方发送响应并计算网络延迟。数据包传输延迟等于`receive_timestamp` - `send_timestamp`。

如果机器之间的系统时钟不一致，Exchange运算符的时间成本就会出现异常。

### 为什么所有运算符的总时间成本明显低于查询执行时间？

可能的原因：在高并发场景中，一些管道驱动程序，尽管是可调度的，但可能因为排队而无法及时处理。等待时间不会记录在运算符的指标中，而是记录在`PendingTime`、`ScheduleTime`和`IOTaskWaitTime`中。

示例：

从配置文件中可以看出，`ExecutionWallTime`大约为55毫秒。然而，所有运算符的总时间成本低于10毫秒，明显低于`ExecutionWallTime`。

![img](../assets/profile-15.jpeg)