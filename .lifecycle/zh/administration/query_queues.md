---
displayed_sidebar: English
---

# 查询队列

本主题介绍如何在 StarRocks 中管理查询队列。

从 v2.5 版本开始，StarRocks 支持查询队列功能。一旦启用查询队列，当并发阈值或资源限制被触达时，StarRocks 将自动对传入的查询请求进行排队，以此避免系统过载。待处理的查询会在队列中等待，直到有足够的计算资源可供执行。自 v3.1.4 版本起，StarRocks 支持在资源组级别设置查询队列。

您可以设置 CPU 使用率、内存使用率和查询并发度的阈值来激活查询队列。

**发展路线图**:

|版本|全局查询队列|资源组级查询队列|集体并发管理|动态并发调整|
|---|---|---|---|---|
|v2.5|✅|❌|❌|❌|
|v3.1.4|✅|✅|✅|✅|

## 启用查询队列

查询队列默认是禁用状态。您可以通过设置相应的全局会话变量来启用全局或资源组级别的查询队列，以便用于 INSERT 加载、SELECT 查询和统计查询。

### 启用全局查询队列

- 为加载任务启用查询队列：

```SQL
SET GLOBAL enable_query_queue_load = true;
```

- 为 SELECT 查询启用查询队列：

```SQL
SET GLOBAL enable_query_queue_select = true;
```

- 为统计查询启用查询队列：

```SQL
SET GLOBAL enable_query_queue_statistic = true;
```

### 启用资源组级别查询队列

从 v3.1.4 版本开始，StarRocks 支持在资源组级别设置查询队列。

要启用资源组级别的查询队列，您需要设置 enable_group_level_query_queue，此外还需设置上述提到的全局会话变量。

```SQL
SET GLOBAL enable_group_level_query_queue = true;
```

## 指定资源阈值

### 为全局查询队列指定资源阈值

您可以通过以下全局会话变量来设置触发查询队列的阈值：

|变量|默认|描述|
|---|---|---|
|query_queue_concurrency_limit|0|BE 上并发查询的上限。设置大于0后才生效。|
|query_queue_mem_used_pct_limit|0|BE上的内存使用百分比上限。设置大于0后才生效。范围：[0, 1]|
|query_queue_cpu_used_permille_limit|0|BE 上的 CPU 使用率 permille 上限（CPU 使用率 * 1000）。设置大于0后才生效。范围：[0, 1000]|

> **注意**
> 默认情况下，BE每隔一秒向FE报告资源使用情况。您可以通过设置BE配置项 `report_resource_usage_interval_ms` 来更改此间隔。

### 为资源组级别查询队列指定资源阈值

从 v3.1.4 版本开始，您可以在创建资源组时，单独设置并发限制（concurrency_limit）和 CPU 核心数限制（max_cpu_cores）。当查询启动时，如果任何资源消耗超过全局或资源组级别的资源阈值，查询将进入队列，直到所有资源消耗都低于阈值。

|变量|默认|描述|
|---|---|---|
|concurrency_limit|0|单个 BE 节点上资源组的并发限制。仅当设置大于0时才生效。|
|max_cpu_cores|0|单个 BE 节点上此资源组的 CPU 核心限制。仅当设置大于0时才生效。范围：[0, avg_be_cpu_cores]，其中avg_be_cpu_cores表示所有BE节点的平均CPU核数。|

您可以使用 SHOW USAGE RESOURCE GROUPS 命令查看每个 BE 节点上各资源组的资源使用情况，详情请参见[查看资源组使用信息](./resource_group.md#view-resource-group-usage-information)部分。

### 管理查询并发

当运行中的查询数量（num_running_queries）超过全局或资源组的并发限制（concurrency_limit）时，新传入的查询将被放入队列中。获取 num_running_queries 的方法在 v3.1.4 之前和之后的版本中有所不同。

- 在 v3.1.4 之前的版本中，BE 会根据 report_resource_usage_interval_ms 配置项指定的间隔报告 num_running_queries。因此，在 num_running_queries 的变化识别上可能会有一些延迟。例如，如果 BE 当前报告的 num_running_queries 没有超过全局或资源组的并发限制，但在下一次报告之前到达的新查询使得并发限制被超过，这些新查询将会被执行，而不用等待在队列中。

- 在 v3.1.4 及以后的版本中，所有运行中的查询都由 Leader FE 集中管理。每个 Follower FE 在启动或完成查询时都会通知 Leader FE，这使得 StarRocks 能够应对查询数量突然增加，超过并发限制的情况。

## 配置查询队列

您可以通过以下全局会话变量来设置查询队列的容量和队列中查询的最大等待时间：

|变量|默认|描述|
|---|---|---|
|query_queue_max_queued_queries|1024|队列中查询的上限。当达到此阈值时，传入的查询将被拒绝。设置大于0后才生效。|
|query_queue_pending_timeout_second|300|队列中待处理查询的最大超时。当达到此阈值时，相应的查询将被拒绝。单位：秒。|

## 配置动态并发调整

从 v3.1.4 版本开始，对于由查询队列管理并由 Pipeline Engine 执行的查询，StarRocks 能够动态调整查询并发度 `pipeline_dop`，以适应当前运行的查询数量 `num_running_queries`、分片数量 `num_fragments` 和查询并发度 `pipeline_dop`。这样可以在最小化调度开销的同时，确保 BE 资源利用最佳，动态控制查询并发度。有关分片和查询并发度 `pipeline_dop` 的更多信息，请参阅[查询管理 - 调整查询并发度](./Query_management.md)。

对于每个在查询队列中的查询，StarRocks 维护了一个“驱动程序（drivers）”的概念，它代表了单个 BE 上查询的并发分片。其逻辑值 num_drivers，即单个 BE 上该查询所有分片的总并发度，等于 num_fragments 乘以 pipeline_dop。当新查询到来时，StarRocks 根据以下规则调整查询并发度 pipeline_dop：

- - 运行中的驱动程序数量（num_drivers）超过并发驱动程序的低水位线（query_queue_driver_low_water）越多，查询并发度 pipeline_dop 调低越多。
- - StarRocks 限制运行中的驱动程序数量（num_drivers）不超过查询并发驱动程序的高水位线（query_queue_driver_high_water）。

您可以使用以下全局会话变量配置查询并发度 pipeline_dop 的动态调整：

|变量|默认|描述|
|---|---|---|
|query_queue_driver_high_water|-1|查询的并发驱动程序的高水位限制。仅当设置为非负值时才生效。当设置为 0 时，相当于 avg_be_cpu_cores * 16，其中 avg_be_cpu_cores 表示所有 BE 节点上的平均 CPU 核心数。当设置为大于 0 的值时，直接使用该值。|
|query_queue_driver_low_water|-1|查询并发驱动程序的下限。仅当设置为非负值时才生效。当设置为0时，相当于avg_be_cpu_cores * 8。当设置为大于0的值时，直接使用该值。|

## 监控查询队列

您可以使用以下方法查看有关查询队列的信息。

### SHOW PROC

您可以使用[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md)命令来检查运行中的查询数量，以及 BE 节点的内存和 CPU 使用情况：

```Plain
mysql> SHOW PROC '/backends'\G
*************************** 1. row ***************************
...
    NumRunningQueries: 0
           MemUsedPct: 0.79 %
           CpuUsedPct: 0.0 %
```

### SHOW PROCESSLIST

您可以使用[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md)命令来检查查询是否在队列中（当 `IsPending` 为 `true` 时）：

```Plain
mysql> SHOW PROCESSLIST;
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
| Id   | User | Host                | Db    | Command | ConnectionStartTime | Time | State | Info              | IsPending |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
|    2 | root | xxx.xx.xxx.xx:xxxxx |       | Query   | 2022-11-24 18:08:29 |    0 | OK    | SHOW PROCESSLIST  | false     |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
```

### FE 审计日志

您可以查看 FE 审计日志文件 **fe.audit.log**。字段 `PendingTimeMs` 表示查询在队列中等待的时间，单位为毫秒。

### FE 指标

以下 FE 指标基于每个 FE 节点的统计数据。

|公制|单位|类型|描述|
|---|---|---|---|
|starrocks_fe_query_queue_pending|计数|瞬时|队列中当前查询的数量。|
|starrocks_fe_query_queue_total|计数|瞬时|历史排队的查询总数（包括当前正在运行的查询）。|
|starrocks_fe_query_queue_timeout|计数|瞬时|队列中超时的查询总数。|
|starrocks_fe_resource_group_query_queue_total|计数|瞬时|历史上在此资源组中排队的查询总数（包括当前正在运行的查询）。名称标签表示资源组的名称。从 v3.1.4 开始支持此指标。|
|starrocks_fe_resource_group_query_queue_pending|计数|瞬时|当前在此资源组的队列中的查询数。名称标签表示资源组的名称。从 v3.1.4 开始支持此指标。|
|starrocks_fe_resource_group_query_queue_timeout|计数|瞬时|此资源组队列中已超时的查询数。名称标签表示资源组的名称。从 v3.1.4 开始支持此指标。|

### SHOW RUNNING QUERIES

从 v3.1.4 版本开始，StarRocks 支持 SHOW RUNNING QUERIES SQL 语句，用于展示每个查询的队列信息。各字段含义如下：

- QueryId：查询的 ID。
- ResourceGroupId：查询命中的资源组的 ID。当没有命中用户定义的资源组时，将显示为“-”。
- StartTime：查询的开始时间。
- PendingTimeout：处于 PENDING 状态的查询在队列中的超时时间。
- QueryTimeout：查询的超时时间。
- State：查询的队列状态，“PENDING”表示查询处于队列中，“RUNNING”表示查询正在执行。
- Slots：查询请求的逻辑资源数量，当前固定为 1。
- Frontend：发起查询的 FE 节点。
- FeStartTime：发起查询的 FE 节点的开始时间。

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
