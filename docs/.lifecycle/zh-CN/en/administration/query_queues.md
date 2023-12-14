displayed_sidebar: "Chinese"
```


# 查询队列

本主题描述了如何在 StarRocks 中管理查询队列。

从v2.5版本开始，StarRocks支持查询队列。启用查询队列后，当达到并发阈值或资源限制时，StarRocks会自动将传入的查询排队，从而避免过载恶化。待处理查询将等待在队列中，直到有足够的计算资源可用于开始执行。从v3.1.4版本开始，StarRocks支持在资源组级别设置查询队列。

您可以设置CPU使用率、内存使用率和查询并发性的阈值以触发查询队列。

**路线图**：

| 版本    | 全局查询队列 | 资源组级别查询队列 | 集体并发管理 | 动态并发调整 |
| ------ | ------------- | ------------------- | ------------- | ------------ |
| v2.5    | ✅            | ❌                   | ❌             | ❌           |
| v3.1.4  | ✅            | ✅                   | ✅             | ✅           |

## 启用查询队列

默认情况下，查询队列已禁用。您可以通过设置相应的全局会话变量为INSERT加载、SELECT查询和统计查询启用全局或资源组级别的查询队列。

### 启用全局查询队列

- 为加载任务启用查询队列：

```SQL
SET GLOBAL enable_query_queue_load = true;
```

- 为SELECT查询启用查询队列：

```SQL
SET GLOBAL enable_query_queue_select = true;
```

- 为统计查询启用查询队列：

```SQL
SET GLOBAL enable_query_queue_statistic = true;
```

### 启用资源组级别查询队列

从v3.1.4版本开始，StarRocks支持在资源组级别设置查询队列。

为了启用资源组级别的查询队列，您还需要设置 `enable_group_level_query_queue`，除了上述提到的全局会话变量。

```SQL
SET GLOBAL enable_group_level_query_queue = true;
```

## 指定资源阈值

### 为全局查询队列指定资源阈值

您可以通过以下全局会话变量设置触发查询队列的阈值：

| **变量**                        | **默认值** | **描述** |
| ------------------------------- | ----------- | ------- |
| query_queue_concurrency_limit   | 0           | 单个BE上并发查询的上限。仅在设置大于 `0` 后生效。 |
| query_queue_mem_used_pct_limit  | 0           | 单个BE上内存使用率的上限百分比。仅在设置大于 `0` 后生效。范围：[0, 1] |
| query_queue_cpu_used_permille_limit | 0        | 单个BE上CPU使用量千分比（CPU使用量 * 1000）的上限。仅在设置大于 `0` 后生效。范围：[0, 1000] |

> **注意**
>
> 默认情况下，BE以一秒为间隔向FE报告资源使用情况。您可以通过设置BE的配置项 `report_resource_usage_interval_ms` 来更改此间隔。

### 为资源组级别查询队列指定资源阈值

从v3.1.4版本开始，您可以在创建资源组时设置各个并发限制 (`concurrency_limit`) 和CPU核心限制 (`max_cpu_cores`)。当启动查询时，如果任何资源消耗超过全局或资源组级别的资源阈值，查询将被放置在队列中，直到所有资源消耗都在阈值范围内。

| **变量**         | **默认值** | **描述** |
| ---------------- | ----------- | ------- |
| concurrency_limit | 0           | 单个BE节点上资源组的并发限制。仅设置为大于 `0` 时生效。 |
| max_cpu_cores    | 0           | 单个BE节点上资源组的CPU核心限制。仅设置为大于 `0` 时生效。范围：[0, `avg_be_cpu_cores`]，其中 `avg_be_cpu_cores` 表示所有BE节点上CPU核心的平均数量。 |

您可以使用 `SHOW RESOURCE GROUPS USAGE` 来查看每个BE节点上每个资源组的资源使用信息，详情请参见 [查看资源组使用信息](./resource_group.md#view-resource-group-usage-information)。

### 管理查询并发

当运行的查询数量 (`num_running_queries`) 超过全局或资源组的 `concurrency_limit` 时，传入的查询将放置在队列中。获取 `num_running_queries` 的方式在版本 `< v3.1.4` 和 `≥ v3.1.4` 之间有所不同。

- 在版本 `< v3.1.4` 中，BE在指定的 `report_resource_usage_interval_ms` 间隔内报告 `num_running_queries`。因此，在确定 `num_running_queries` 的变化时可能会有一些延迟。例如，如果BE在当前时刻报告的 `num_running_queries` 不超过全局或资源组的 `concurrency_limit`，但在下一次报告之前传入的查询到达并超过 `concurrency_limit`，这些传入的查询将在不等待队列中执行。

- 在版本 `≥ v3.1.4` 中，所有运行中的查询由Leader FE集体管理。每个Follower FE在启动或完成查询时都会通知Leader FE，这使得StarRocks能够处理突然增加的查询超过 `concurrency_limit` 的情况。

## 配置查询队列

您可以通过以下全局会话变量来设置查询队列的容量和队列中查询的最大超时时间：

| **变量**                          | **默认值** | **描述** |
| --------------------------------- | ----------- | ------- |
| query_queue_max_queued_queries    | 1024        | 队列中的查询上限。当达到此阈值时，传入的查询将被拒绝。仅在设置大于 `0` 后生效。 |
| query_queue_pending_timeout_second | 300         | 队列中待处理查询的最大超时时间。当达到此阈值时，相应的查询将被拒绝。单位：秒。 |

## 配置查询并发的动态调整

从v3.1.4版本开始，对由查询队列管理并由Pipeline Engine运行的查询，StarRocks可以根据当前运行的查询数量 `num_running_queries`、片数 `num_fragments` 和查询并发 `pipeline_dop` 来动态调整查询并发。这使您可以动态控制查询并发，同时最大程度地减少调度开销，确保BE资源利用率最优。有关片数和查询并发 `pipeline_dop` 的更多信息，请参见 [查询管理 - 调整查询并发](./Query_management.md)。

对于查询队列下的每个查询，StarRocks维护了驱动程序的概念，这些驱动程序代表了单个BE上查询的并发片段。其逻辑值 `num_drivers`，代表了该查询在单个BE上所有片段的总并发，等于 `num_fragments * pipeline_dop`。新查询到达时，StarRocks根据以下规则调整查询并发 `pipeline_dop`：

- `num_drivers` 的运行数量越高超过并发驱动程序的低水位限制 `query_queue_driver_low_water`，查询并发 `pipeline_dop` 被调整得越低。
- StarRocks限制了查询的运行驱动程序数量 `num_drivers` 不超过查询并发驱动程序的高水位限制。

您可以使用以下全局会话变量配置查询并发 `pipeline_dop` 的动态调整：

| **变量**                  | **默认值** | **描述** |
| ------------------------- | ----------- | ------- |
| query_queue_driver_high_water | -1          | 查询的并发驱动程序的高水位限制。仅在设置为非负值时生效。当设置为 `0` 时，等效于 `avg_be_cpu_cores * 16`，其中 `avg_be_cpu_cores` 表示所有BE节点上CPU核心的平均数量。当设置为大于 `0` 的值时，直接使用该值。 |
| query_queue_driver_low_water  | -1          | 查询的并发驱动程序的低水位限制。仅在设置为非负值时生效。当设置为 `0` 时，等效于 `avg_be_cpu_cores * 8`。当设置为大于 `0` 的值时，直接使用该值。 |

## 监视查询队列

您可以使用以下方法查看与查询队列相关的信息。

### `SHOW PROC`

您可以使用 [SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md) 来查看BE节点上运行查询的数量以及内存和CPU的使用情况：

```Plain
mysql> SHOW PROC '/backends'\G
*************************** 1. row ***************************
...
    NumRunningQueries: 0
           MemUsedPct: 0.79 %
           CpuUsedPct: 0.0 %
```

### `SHOW PROCESSLIST`

您可以使用 [SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) 来检查查询是否在队列中（当 `IsPending` 为 `true` 时）：

```Plain
mysql> SHOW PROCESSLIST;
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
| Id   | User | Host                | Db    | Command | ConnectionStartTime | Time | State | Info              | IsPending |
```markdown
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
```


### FE审计日志

您可以查看FE审计日志文件**fe.audit.log**。字段`PendingTimeMs`指示查询在队列中等待的时间，其单位为毫秒。

### FE指标

以下FE指标来源于每个FE节点的统计数据。

| 指标                                           | 单位 | 类型    | 描述                                                     |
| ----------------------------------------------- | ---- | ------- | ---------------------------------------------------------- |
| starrocks_fe_query_queue_pending                | 计数 | 瞬时    | 队列中当前的查询数量。                                       |
| starrocks_fe_query_queue_total                  | 计数 | 瞬时    | 历史上排队的查询总数（包括当前正在运行的查询）。              |
| starrocks_fe_query_queue_timeout                | 计数 | 瞬时    | 在队列中超时的查询总数。                                      |
| starrocks_fe_resource_group_query_queue_total   | 计数 | 瞬时    | 在此资源组中历史上排队的查询总数（包括当前正在运行的查询）。`name`标签表示资源组的名称。此指标从v3.1.4版本开始支持。 |
| starrocks_fe_resource_group_query_queue_pending | 计数 | 瞬时    | 此资源组队列中当前的查询数量。`name`标签表示资源组的名称。此指标从v3.1.4版本开始支持。 |
| starrocks_fe_resource_group_query_queue_timeout | 计数 | 瞬时    | 在此资源组队列中超时的查询总数。`name`标签表示资源组的名称。此指标从v3.1.4版本开始支持。 |

### 显示运行中的查询

从v3.1.4开始，StarRocks支持SQL语句`SHOW RUNNING QUERIES`，用于显示每个查询的队列信息。每个字段的含义如下：

- `QueryId`: 查询的ID。
- `ResourceGroupId`: 查询命中的资源组的ID。当未命中用户定义的资源组时，将显示为“-”。
- `StartTime`: 查询的开始时间。
- `PendingTimeout`: PENDING查询将在队列中超时的时间。
- `QueryTimeout`: 查询超时的时间。
- `State`: 查询的队列状态，“PENDING”表示它在队列中，“RUNNING”表示当前正在执行。
- `Slots`: 查询请求的逻辑资源数量，目前固定为`1`。
- `Frontend`: 发起查询的FE节点。
- `FeStartTime`: 发起查询的FE节点的开始时间。

示例：

```Plain
MySQL [(none)]> SHOW RUNNING QUERIES;
+--------------------------------------+-----------------+---------------------+---------------------+---------------------+-----------+-------+---------------------------------+---------------------+
| QueryId                              | ResourceGroupId | StartTime           | PendingTimeout      | QueryTimeout        |   State   | Slots | Frontend                        | FeStartTime         |
+--------------------------------------+-----------------+---------------------+---------------------+---------------------+-----------+-------+---------------------------------+---------------------+
| a46f68c6-3b49-11ee-8b43-00163e10863a | -               | 2023-08-15 16:56:37 | 2023-08-15 17:01:37 | 2023-08-15 17:01:37 |  RUNNING  | 1     | 127.00.00.01_9010_1692069711535 | 2023-08-15 16:37:03 |
| a6935989-3b49-11ee-935a-00163e13bca3 | 12003           | 2023-08-15 16:56:40 | 2023-08-15 17:01:40 | 2023-08-15 17:01:40 |  RUNNING  | 1     | 127.00.00.02_9010_1692069658426 | 2023-08-15 16:37:03 |
| a7b5e137-3b49-11ee-8b43-00163e10863a | 12003           | 2023-08-15 16:56:42 | 2023-08-15 17:01:42 | 2023-08-15 17:01:42 |  PENDING  | 1     | 127.00.00.03_9010_1692069711535 | 2023-08-15 16:37:03 |
+--------------------------------------+-----------------+---------------------+---------------------+---------------------+-----------+-------+---------------------------------+---------------------+
```