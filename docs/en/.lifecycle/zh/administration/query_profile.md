---
displayed_sidebar: English
---

# 分析查询配置文件

本主题描述了如何检查查询配置文件。查询配置文件记录了查询中涉及的所有工作节点的执行信息。查询配置文件帮助您快速识别影响 StarRocks 集群查询性能的瓶颈。

## 启用查询配置文件

对于早于 v2.5 的 StarRocks 版本，您可以通过将变量 `is_report_success` 设置为 `true` 来启用查询配置文件：

```SQL
SET is_report_success = true;
```

对于 StarRocks v2.5 或更新版本，您可以通过将变量 `enable_profile` 设置为 `true` 来启用查询配置文件：

```SQL
SET enable_profile = true;
```

### 运行时配置文件

从 v3.1 开始，StarRocks 支持运行时配置文件功能，使您能够在查询完成之前访问查询配置文件。

要使用此功能，除了将 `enable_profile` 设置为 `true` 外，还需要设置会话变量 `runtime_profile_report_interval`。`runtime_profile_report_interval`（单位：秒，默认值：`10`）用于控制配置文件报告间隔，默认设置为 10 秒，这意味着每当查询时间超过 10 秒时，运行时配置文件功能会自动启用。

```SQL
SET runtime_profile_report_interval = 10;
```

运行时配置文件显示的信息与任何查询配置文件显示的信息相同。您可以像分析常规查询配置文件一样对其进行分析，以获得有关正在运行的查询性能的宝贵见解。

但是，运行时配置文件可能不完整，因为执行计划的某些运算符可能依赖于其他运算符。为了便于区分正在运行的运算符和已完成的运算符，正在运行的运算符标有 `Status: Running`。

## 访问查询配置文件

> **注意**
>
> 如果您使用的是 StarRocks 企业版，则可以使用 StarRocks Manager 访问和可视化您的查询配置文件。

如果您使用的是 StarRocks 社区版，请按照以下步骤访问您的查询配置文件：

1. 在浏览器中输入 `http://<fe_ip>:<fe_http_port>`。
2. 在显示的页面上，单击顶部导航窗格中的 **查询**。
3. 在“已完成的查询”列表中，选择要检查的查询，然后单击“配置文件”列中的链接。

![图片](../assets/profile-1.png)

浏览器会将您重定向到包含相应查询配置文件的新页面。

![图片](../assets/profile-2.png)

## 解释查询配置文件

### 查询配置文件的结构

以下是一个示例查询配置文件：

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
- 流水线：执行链。执行链没有分支。一个片段可以拆分为多个管道。
- 运算符：管道由多个运算符组成。

![图片](../assets/profile-3.png)

*一个片段由多个管道组成。*

### 关键指标

查询配置文件包含大量指标，这些指标显示查询执行的详细信息。在大多数情况下，您只需要观察算子的执行时间和它们处理的数据的大小。找到瓶颈后，您可以相应地解决它们。

#### 总结

| 度量       | 描述                                                  |
| ------------ | ------------------------------------------------------------ |
| 总        | 查询消耗的总时间，包括计划、执行和分析所花费的时间。 |
| 查询CpuCost | 查询的总 CPU 时间成本。CPU 时间成本会聚合并发进程。因此，此指标的值可能大于查询的实际执行时间。 |
| QueryMemCost | 查询的总内存成本。                              |

#### 运营商的通用指标

| 度量            | 描述                                           |
| ----------------- | ----------------------------------------------------- |
| 运算符TotalTime | 运算符的总 CPU 时间成本。                  |
| PushRowNum        | 运算符推送的数据的总行数。 |
| 拉行数        | 运算符拉取的数据的总行数。 |

#### 唯一指标

| 度量            | 描述                             |
| ----------------- | --------------------------------------- |
| IOTaskExecTime（物联网：AskExecTime）    | 所有 I/O 任务的总执行时间。 |
| IOTaskWaitTime（物联网等待时间）    | 所有 I/O 任务的总等待时间。      |
| Morsels计数      | I/O 任务总数。              |

#### 扫描运算符

| 度量                          | 描述                                                  |
| ------------------------------- | ------------------------------------------------------------ |
| 桌子                           | 表名。                                                  |
| 扫描时间                        | 总扫描时间。扫描在异步 I/O 线程池中执行。 |
| 平板电脑计数                     | 片剂数量。                                           |
| 下推谓词              | 下推的谓词数。              |
| 字节读取                       | StarRocks 读取的数据大小。                          |
| 压缩字节读取             | StarRocks 读取的压缩数据的大小。               |
| 物联网                          | 总 I/O 时间。                                              |
| BitmapIndexFilterRows           | 按位图索引筛选出的数据的行计数。 |
| BloomFilterFilterRows           | 由 Bloomfilter 筛选掉的数据的行计数。 |
| SegmentRuntimeZoneMapFilterRows | 按运行时区域映射筛选出的数据的行计数。 |
| SegmentZoneMapFilterRows        | 按区域映射筛选出的数据的行计数。    |
| ShortKeyFilter行              | 由短键筛选出的数据的行数。   |
| ZoneMapIndexFilterRows          | 按区域映射索引筛选出的数据的行计数。 |

#### 交易所运营商

| 度量            | 描述                                                  |
| ----------------- | ------------------------------------------------------------ |
| 零件类型          | 数据分布类型。有效值： `UNPARTITIONED` 、 `RANDOM`、 `HASH_PARTITIONED` `BUCKET_SHUFFLE_HASH_PARTITIONED`和 。 |
| 字节发送         | 发送的数据的大小。                             |
| 总体吞吐量 | 总体吞吐量。                                          |
| 网络时间       | 数据包传输时间（不包括接收后处理的时间）。请参阅下面的常见问题解答，详细了解此指标的计算方式以及如何遇到异常。 |
| 等待时间          | 等待时间，因为发送方的队列已满。   |

#### 聚合运算符

| 度量             | 描述                                   |
| ------------------ | --------------------------------------------- |
| 分组键       | 分组键的名称（GROUP BY 列）。 |
| 聚合函数 | 聚合函数。                          |
| AggComputeTime（阿格计算时间）     | 聚合函数消耗的计算时间。 |
| ExprComputeTime（Expr计算时间）    | 表达式消耗的计算时间。      |
| 哈希表大小      | 哈希表的大小。                       |

#### Join 运算符

| 度量                    | 描述                         |
| ------------------------- | ----------------------------------- |
| JoinPredicates            | JOIN 操作的谓词。   |
| JoinType                  | JOIN 类型。                      |
| 构建存储桶              | 哈希表的存储桶编号。    |
| BuildHashTableTime（构建哈希表时间）        | 用于生成哈希表的时间。  |
| ProbeConjunctEvaluateTime | Probe Conjunct 消耗的时间。    |
| SearchHashTableTimer      | 用于搜索哈希表的时间。 |

#### Window Function 运算符

| 度量             | 描述                                           |
| ------------------ | ----------------------------------------------------- |
| 计算时间        | 窗口函数消耗的计算时间。         |
| 分区键      | 分区键的名称（PARTITION BY 列）。 |
| 聚合函数 | 聚合函数。                                  |

#### 排序运算符

| 度量   | 描述                                                  |
| -------- | ------------------------------------------------------------ |
| 排序键 | 排序键的名称（ORDER BY 列）。                    |
| 排序类型 | 结果排序类型：列出所有结果，或列出前 n 个结果。 |

#### TableFunction 运算符

| 度量                 | 描述                                     |
| ---------------------- | ----------------------------------------------- |
| 表函数ExecTime  | 表函数消耗的计算时间。    |
| 表函数ExecCount | 表函数的执行次数。 |

#### 项目运营商

| 度量                   | 描述                                         |
| ------------------------ | --------------------------------------------------- |
| ExprComputeTime（Expr计算时间）          | 表达式消耗的计算时间。            |
| CommonSubExprComputeTime（通用子Expr计算时间） | 公共子表达式消耗的计算时间。 |

#### LocalExchange 运算符

| 度量     | 描述                                                  |
| ---------- | ------------------------------------------------------------ |
| 类型       | 本地 Exchange 类型。有效值： `Passthrough`、 `Partition`和 `Broadcast`。 |
| 随机播放 | 洗牌次数。仅当 `Type` 为 `Partition` 时，此指标才有效。 |

#### Hive 连接器

| 度量                      | 描述                                            |
| --------------------------- | ------------------------------------------------------ |
| 扫描范围                  | 扫描的平板电脑数量。                    |
| ReaderInit（读者初始化）                  | Reader 的启动时间。                         |
| 列读取时间              | Reader 读取和解析数据所花费的时间。        |
| ExprFilterTime（ExprFilterTime）              | 用于筛选表达式的时间。                       |
| 行读取                    | 读取的数据行数。                     |

#### 输入流

| 度量                    | 描述                                                               |
| ------------------------- | ------------------------------------------------------------------------- |
| AppIOBytes读取            | I/O 任务从应用层读取的数据大小。            |
| AppIOCounter              | 来自应用层的 I/O 任务数。                           |
| AppIOTime                 | 应用层的 I/O 任务读取数据所消耗的总时间。 |
| FSBytes读取               | 存储系统读取的数据的大小。                              |
| FSIO控制器               | 存储层的 I/O 任务数。                               |
| FSIOTime                  | 存储层读取数据所消耗的总时间。                    |

### 操作员消耗的时间

- 对于 OlapScan 和 ConnectorScan 运算符，它们的时间消耗相当于 `OperatorTotalTime + ScanTime`。由于 Scan 运算符在异步 I/O 线程池中执行 I/O 操作，因此 ScanTime 表示异步 I/O 时间。
- Exchange 运算符的时间消耗相当于 `OperatorTotalTime + NetworkTime`。由于 Exchange 操作员在 bRPC 线程池中发送和接收数据包，因此 NetworkTime 表示网络传输所消耗的时间。
- 对于所有其他运营商，他们的时间成本是 `OperatorTotalTime`。

### 指标合并和最小值/最大值

流水线引擎是一种并行计算引擎。每个 Fragment 分发到多台机器进行并行处理，每台机器上的流水线作为多个并发实例并行执行。因此，在分析时，StarRocks 会合并相同的指标，并记录所有并发实例中每个指标的最小值和最大值。


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

- 非时间指标进行汇总。例如：
  - `PullChunkNum` 表示所有并发实例的总数。
  - `__MAX_OF_PullChunkNum` 是所有并发实例中的最大值。
  - `__MIN_OF_PullChunkNum` 是所有并发实例中的最小值。

  ```SQL
                 - PullChunkNum: 146.66K (146660)
                   - __MAX_OF_PullChunkNum: 24.45K (24450)
                   - __MIN_OF_PullChunkNum: 24.435K (24435)
  ```

- 一些特殊指标没有最小值和最大值，在所有并发实例中具有相同的值（例如， `DegreeOfParallelism`）。

#### MIN 和 MAX 之间的显著差异

通常，MIN 和 MAX 值之间的显著差异表明数据存在偏斜。这可能发生在聚合或 JOIN 操作期间。

```SQL
             - OperatorTotalTime: 2m48s
               - __MAX_OF_OperatorTotalTime: 10m30s
               - __MIN_OF_OperatorTotalTime: 279.170us
```

## 执行基于文本的配置文件分析

从 v3.1 开始，StarRocks 提供了更加用户友好的基于文本的配置文件分析功能。此功能使您能够高效地识别查询优化的瓶颈和机会。

### 分析现有查询

您可以通过其 `QueryID` 分析现有查询的配置文件，无论查询是正在运行还是已完成。

#### 列出配置文件

执行以下 SQL 语句以列出现有配置文件：

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

此 SQL 语句允许您轻松获取每个查询关联的 `QueryId`。`QueryId` 是进一步剖面分析和调查的关键标识符。

#### 分析配置文件

获得 `QueryId` 后，您可以使用 ANALYZE PROFILE 语句对特定查询执行更详细的分析。此 SQL 语句提供更深入的见解，并有助于全面检查查询的性能特征和优化。

```sql
ANALYZE PROFILE FROM '<QueryId>' [, <plan_node_id>, ...]
```

默认情况下，分析输出仅显示每个运算符的最关键指标。但是，您可以指定一个或多个计划节点的 ID 来查看相应的指标。此功能允许更全面地检查查询的性能，并有助于有针对性的优化。

示例 1：在不指定计划节点 ID 的情况下分析配置文件：

![img](../assets/profile-16.png)

示例 2：分析配置文件并指定计划节点的 ID：

![img](../assets/profile-17.png)

ANALYZE PROFILE 语句提供了一种用于分析和理解运行时配置文件的增强方法。它区分具有不同状态的运算符，例如“已阻止”、“正在运行”和“已完成”。此外，该语句提供了整个进度以及基于处理的行号的单个运算符的进度的全面视图，从而可以更深入地了解查询执行和性能。该功能进一步方便了 StarRocks 中查询的分析和优化。

示例 3：分析正在运行的查询的运行时配置文件：

![img](../assets/profile-20.png)

### 分析模拟查询

您还可以使用 EXPLAIN ANALYZE 语句模拟给定查询并分析其配置文件。

```sql
EXPLAIN ANALYZE <sql>
```

目前，EXPLAIN ANALYZE 支持两种类型的 SQL 语句：查询 （SELECT） 语句和 INSERT INTO 语句。您只能模拟 INSERT INTO 语句，并在 StarRocks 的原生 OLAP 表上分析其配置文件。请注意，当您分析 INSERT INTO 语句的配置文件时，实际上不会插入任何数据。默认情况下，事务处于中止状态，确保在配置文件分析过程中不会对数据进行意外更改。

示例 1：分析给定查询的配置文件：

![img](../assets/profile-18.png)

示例 1：分析 INSERT INTO 操作的配置文件：

![img](../assets/profile-19.png)

## 可视化查询配置文件

如果您是 StarRocks 企业版用户，您可以通过 StarRocks Manager 可视化您的查询配置文件。

“配置文件概览”页面显示一些汇总指标，包括总执行时间 `ExecutionWallTime`、I/O 指标、网络传输大小以及 CPU 和 I/O 时间占比。

![img](../assets/profile-4.jpeg)

通过单击操作员（节点）的卡片，您可以在页面的右侧窗格中查看其详细信息。有三个选项卡：

- **Node**：该算子的核心指标。
- **Node Detail**：该算子的所有指标。
- **Pipeline**：算子所属流水线的指标。您无需过多关注此选项卡，因为它仅与计划相关。

![img](../assets/profile-5.jpeg)

### 识别瓶颈

操作员花费的时间比例越大，其卡片的颜色就越深。这有助于您轻松识别查询的瓶颈。

![img](../assets/profile-6.jpeg)

### 检查数据是否偏斜

单击占用大量时间的运算符的卡片，然后检查其 `MaxTime` 和 `MinTime`。`MaxTime` 和 `MinTime` 之间的明显差异通常表示数据存在偏斜。

![img](../assets/profile-7.jpeg)

然后，单击“节点详细信息”选项卡，并检查是否有任何指标显示异常。在此示例中，Aggregate 运算符的 `PushRowNum` 指标显示数据倾斜。

![img](../assets/profile-8.jpeg)

### 检查分区或分桶策略是否生效

您可以通过查看对应的查询计划来检查分区或分桶策略是否生效 `EXPLAIN <sql_statement>`。

![img](../assets/profile-9.png)

### 检查是否使用了正确的物化视图

单击相应的 Scan 运算符，然后检查“Node Detail”选项卡上的 `Rollup` 字段。

![img](../assets/profile-10.jpeg)

### 检查 JOIN 计划是否适用于左表和右表

通常，StarRocks 会选择较小的表作为 Join 的右表。如果查询配置文件显示其他情况，则会发生异常。

![img](../assets/profile-11.jpeg)

### 检查 JOIN 的分发类型是否正确

Exchange 运算符根据数据分布类型分为三种类型：

- `UNPARTITIONED`：广播。数据被制作成多个副本并分发到多个 BE。
- `RANDOM`：轮询。
- `HASH_PARTITIONED` 和 `BUCKET_SHUFFLE_HASH_PARTITIONED`：随机播放。`HASH_PARTITIONED` 和 `BUCKET_SHUFFLE_HASH_PARTITIONED` 之间的区别在于用于计算哈希代码的哈希函数。

对于内部连接，右侧表可以是 `HASH_PARTITIONED` 和 `BUCKET_SHUFFLE_HASH_PARTITIONED` 类型或 `UNPARTITIONED` 类型。通常，只有当右表中的行数少于 100K 时，才会采用 `UNPARTITIONED` 类型。

在以下示例中，Exchange 算子的类型为 Broadcast，但算子传输的数据量大大超过阈值。

![img](../assets/profile-12.jpeg)

### 检查 JoinRuntimeFilter 是否生效

当 Join 的右子树正在构建哈希表时，它会创建一个运行时筛选器。此运行时筛选器将发送到左子树，并在可能的情况下向下推送到 Scan 运算符。您可以在 Scan 运算符的“Node Detail”选项卡上检查与 `JoinRuntimeFilter` 相关的指标。

![img](../assets/profile-13.jpeg)

## 常见问题

### 为什么 Exchange 运算符的时间成本异常？

![img](../assets/profile-14.jpeg)

Exchange 运算符的时间成本由两部分组成：CPU 时间和网络时间。网络时间依赖于系统时钟。网络时间的计算方法如下：

1. 发送方在调用 bRPC 接口发送包之前记录 `send_timestamp`。
2. 接收方在收到来自 bRPC 接口的包后记录 `receive_timestamp`（不包括接收后处理时间）。
3. 处理完成后，接收方发送响应并计算网络延迟。包传输延迟相当于 `receive_timestamp` - `send_timestamp`。

如果计算机之间的系统时钟不一致，则 Exchange 运算符的时间成本将发生异常。


### 为什么所有操作者的总时间成本明显小于查询执行时间？

可能原因：在高并发场景中，一些流水线驱动程序虽然可调度，但由于排队，可能无法及时处理。等待时间不记录在操作者的指标中，而是记录在 `PendingTime`、 `ScheduleTime` 和 `IOTaskWaitTime` 中。

示例：

从配置文件中，我们可以看到 `ExecutionWallTime` 大约是 55 毫秒。然而，所有操作者的总时间成本小于 10 毫秒，明显低于 `ExecutionWallTime`。

![图片](../assets/profile-15.jpeg)