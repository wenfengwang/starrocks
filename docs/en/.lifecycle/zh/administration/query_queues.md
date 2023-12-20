---
displayed_sidebar: English
---

# 查询队列

本主题介绍如何在 StarRocks 中管理查询队列。

从 v2.5 版本开始，StarRocks 支持查询队列功能。启用查询队列后，当系统达到并发阈值或资源限制时，StarRocks 会自动将传入的查询排队，以此避免系统过载。待处理的查询会在队列中等待，直到有足够的计算资源可用来开始执行。从 v3.1.4 版本起，StarRocks 支持在资源组级别设置查询队列。

您可以设置 CPU 使用率、内存使用率和查询并发数的阈值来触发查询队列。

**发展路线图**：

|版本|全局查询队列|资源组级查询队列|集体并发管理|动态并发调整|
|---|---|---|---|---|
|v2.5|✅|❌|❌|❌|
|v3.1.4|✅|✅|✅|✅|

## 启用查询队列

查询队列默认是禁用的。您可以通过设置相应的全局会话变量来启用全局或资源组级别的查询队列，以便于 INSERT 加载、SELECT 查询和统计查询。

### 启用全局查询队列

- 启用加载任务的查询队列：

```SQL
SET GLOBAL enable_query_queue_load = true;
```

- 启用 SELECT 查询的查询队列：

```SQL
SET GLOBAL enable_query_queue_select = true;
```

- 启用统计查询的查询队列：

```SQL
SET GLOBAL enable_query_queue_statistic = true;
```

### 启用资源组级查询队列

从 v3.1.4 版本开始，StarRocks 支持在资源组级别设置查询队列。

要启用资源组级别的查询队列，除了上述提到的全局会话变量之外，您还需要设置 `enable_group_level_query_queue`。

```SQL
SET GLOBAL enable_group_level_query_queue = true;
```

## 指定资源阈值

### 指定全局查询队列的资源阈值

您可以通过以下全局会话变量设置触发查询队列的阈值：

|**变量**|**默认值**|**描述**|
|---|---|---|
|query_queue_concurrency_limit|0|BE 上并发查询的上限。仅当设置值大于 `0` 时才生效。|
|query_queue_mem_used_pct_limit|0|BE 上的内存使用百分比上限。仅当设置值大于 `0` 时才生效。范围：[0, 1]|
|query_queue_cpu_used_permille_limit|0|BE 上的 CPU 使用率千分比上限（CPU 使用率 * 1000）。仅当设置值大于 `0` 时才生效。范围：[0, 1000]|

> **注意**
> 默认情况下，BE 每隔一秒向 FE 报告资源使用情况。您可以通过设置 BE 配置项 `report_resource_usage_interval_ms` 来更改此间隔。

### 指定资源组级别查询队列的资源阈值

从 v3.1.4 版本开始，您可以在创建资源组时单独设置并发限制 (`concurrency_limit`) 和 CPU 核心限制 (`max_cpu_cores`)。当查询启动时，如果任何资源消耗超过全局或资源组级别的资源阈值，该查询将被放入队列中，直到所有资源消耗都在阈值之内。

|**变量**|**默认值**|**描述**|
|---|---|---|
|concurrency_limit|0|单个 BE 节点上资源组的并发限制。仅当设置值大于 `0` 时才生效。|
|max_cpu_cores|0|单个 BE 节点上该资源组的 CPU 核心限制。仅当设置值大于 `0` 时才生效。范围：[0, `avg_be_cpu_cores`]，其中 `avg_be_cpu_cores` 表示所有 BE 节点的平均 CPU 核心数。|

您可以使用 SHOW USAGE RESOURCE GROUPS 来查看每个 BE 节点上每个资源组的资源使用情况，具体方法请参见[查看资源组使用信息](./resource_group.md#view-resource-group-usage-information)。

### 管理查询并发

当正在运行的查询数量 (`num_running_queries`) 超过全局或资源组的 `concurrency_limit` 时，传入的查询将被放入队列。获取 `num_running_queries` 的方式在版本小于 v3.1.4 和版本大于等于 v3.1.4 之间有所不同。

- 在版本小于 v3.1.4 中，BE 会按照 `report_resource_usage_interval_ms` 中指定的时间间隔报告 `num_running_queries`。因此，在识别 `num_running_queries` 的变化时可能会有一些延迟。例如，如果 BE 当前报告的 `num_running_queries` 没有超过全局或资源组的 `concurrency_limit`，但在下一次报告之前到达的传入查询超过了 `concurrency_limit`，这些传入的查询将被执行，而不需要在队列中等待。

- 在版本大于等于 v3.1.4 中，所有正在运行的查询都由 Leader FE 集中管理。每个 Follower FE 在启动或完成查询时都会通知 Leader FE，这使得 StarRocks 能够处理查询数量突然增加并超过 `concurrency_limit` 的情况。

## 配置查询队列

您可以通过以下全局会话变量设置查询队列的容量和队列中查询的最大超时时间：

|**变量**|**默认值**|**描述**|
|---|---|---|
|query_queue_max_queued_queries|1024|队列中查询的上限数量。当达到此阈值时，新传入的查询将被拒绝。仅当设置值大于 `0` 时才生效。|
|query_queue_pending_timeout_second|300|队列中待处理查询的最大超时时间。当达到此阈值时，相应的查询将被拒绝。单位：秒。|

## 配置查询并发的动态调整

从 v3.1.4 版本开始，对于由查询队列管理并由 Pipeline Engine 执行的查询，StarRocks 可以根据当前正在运行的查询数量 `num_running_queries`、分片数量 `num_fragments` 和查询并发 `pipeline_dop` 动态调整查询并发。这允许您在最小化调度开销的同时动态控制查询并发，确保 BE 资源的最佳利用。有关分片和查询并发 `pipeline_dop` 的更多信息，请参见[查询管理 - 调整查询并发](./Query_management.md)。

对于每个在查询队列下的查询，StarRocks 维护了一个驱动程序的概念，它代表了单个 BE 上查询的并发分片。其逻辑值 `num_drivers`，代表了该查询在单个 BE 上所有分片的总并发数，等于 `num_fragments * pipeline_dop`。当新查询到来时，StarRocks 根据以下规则调整查询并发 `pipeline_dop`：

- 正在运行的驱动程序数量 `num_drivers` 超过并发驱动程序的低水位限制 `query_queue_driver_low_water` 越多，查询并发 `pipeline_dop` 调整得越低。
- StarRocks 限制正在运行的驱动程序数量 `num_drivers` 低于查询的并发驱动程序高水位限制 `query_queue_driver_high_water`。

您可以使用以下全局会话变量配置查询并发 `pipeline_dop` 的动态调整：

|**变量**|**默认值**|**描述**|
|---|---|---|
|query_queue_driver_high_water|-1|查询的并发驱动程序的高水位限制。仅当设置为非负值时才生效。当设置为 `0` 时，相当于 `avg_be_cpu_cores * 16`，其中 `avg_be_cpu_cores` 表示所有 BE 节点的平均 CPU 核心数。当设置为大于 `0` 的值时，直接使用该值。|
|query_queue_driver_low_water|-1|查询的并发驱动程序的低水位限制。仅当设置为非负值时才生效。当设置为 `0` 时，相当于 `avg_be_cpu_cores * 8`。当设置为大于 `0` 的值时，直接使用该值。|

## 监控查询队列

您可以使用以下方法查看与查询队列相关的信息。

### SHOW PROC

您可以使用 [SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md) 检查 BE 节点中的运行查询数量以及内存和 CPU 使用情况：

```Plain
mysql> SHOW PROC '/backends'\G
*************************** 1. 行 ***************************
...
    NumRunningQueries: 0
           MemUsedPct: 0.79 %
           CpuUsedPct: 0.0 %
```

### SHOW PROCESSLIST

您可以使用 [SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) 检查查询是否在队列中（当 `IsPending` 为 `true` 时）：

```Plain
mysql> SHOW PROCESSLIST;
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
| Id   | User | Host                | Db    | Command | ConnectionStartTime | Time | State | Info              | IsPending |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
|    2 | root | xxx.xx.xxx.xx:xxxxx |       | Query   | 2022-11-24 18:08:29 |    0 | OK    | SHOW PROCESSLIST  | false     |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
```

### FE 审计日志

您可以检查 FE 审计日志文件 **fe.audit.log**。字段 `PendingTimeMs` 表示查询在队列中等待的时间，单位是毫秒。

### FE 指标

以下 FE 指标来自每个 FE 节点的统计数据。

|指标|单位|类型|描述|
|---|---|---|---|
|starrocks_fe_query_queue_pending|计数|瞬时|当前队列中的查询数量。|
|starrocks_fe_query_queue_total|计数|瞬时|历史上队列中的查询总数（包括当前运行的）。|
|starrocks_fe_query_queue_timeout|计数|瞬时|在队列中超时的查询总数。|
|starrocks_fe_resource_group_query_queue_total|计数|瞬时|该资源组历史上队列中的查询总数（包括当前运行的）。`name` 标签表示资源组的名称。此指标从 v3.1.4 版本开始支持。|
|starrocks_fe_resource_group_query_queue_pending|计数|瞬时|当前该资源组队列中的查询数量。`name` 标签表示资源组的名称。此指标从 v3.1.4 版本开始支持。|
|starrocks_fe_resource_group_query_queue_timeout|计数|瞬时|该资源组队列中超时的查询数量。`name` 标签表示资源组的名称。此指标从 v3.1.4 版本开始支持。|

### SHOW RUNNING QUERIES

从 v3.1.4 版本开始，StarRocks 支持 `SHOW RUNNING QUERIES` SQL 语句，用于显示每个查询的队列信息。每个字段的含义如下：

- `QueryId`：查询的 ID。
- `ResourceGroupId`：查询命中的资源组 ID。当没有命中用户定义的资源组时，将显示为 "-"。
- `StartTime`：查询的开始时间。
- `PendingTimeout`：队列中 PENDING 查询的超时时间。
- `QueryTimeout`：查询的超时时间。
- `State`：查询的队列状态，其中 "PENDING" 表示它在队列中，"RUNNING" 表示它正在执行。
- `Slots`：查询请求的逻辑资源量，目前固定为 `1`。
- `Frontend`：发起查询的 FE 节点。
- `FeStartTime`：发起查询的 FE 节点的开始时间。

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
```
对于每个查询队列中的查询，StarRocks 维护了一个驱动程序（drivers）的概念，代表了单个 BE 上查询的并发片段。其逻辑值 `num_drivers` 表示单个 BE 上该查询所有片段的总并发数，等于 `num_fragments * pipeline_dop`。当新查询到来时，StarRocks 根据以下规则调整查询并发 `pipeline_dop`：

- 运行的驱动程序数量 `num_drivers` 超过并发驱动程序的低水位限制 `query_queue_driver_low_water` 越多，查询并发 `pipeline_dop` 调整得越低。
- StarRocks 将正在运行的驱动程序数量 `num_drivers` 限制在查询的并发驱动程序高水位限制 `query_queue_driver_high_water` 以下。

您可以使用以下全局会话变量配置查询并发 `pipeline_dop` 的动态调整：

|**变量**|**默认**|**描述**|
|---|---|---|
|query_queue_driver_high_water|-1|查询的并发驱动程序的高水位限制。仅当设置为非负值时才生效。当设置为 `0` 时，相当于 `avg_be_cpu_cores * 16`，其中 `avg_be_cpu_cores` 表示所有 BE 节点上的平均 CPU 核心数。当设置为大于 `0` 的值时，直接使用该值。|
|query_queue_driver_low_water|-1|查询并发驱动程序的低水位限制。仅当设置为非负值时才生效。当设置为 `0` 时，相当于 `avg_be_cpu_cores * 8`。当设置为大于 `0` 的值时，直接使用该值。|

## 监控查询队列

您可以使用以下方法查看与查询队列相关的信息。

### SHOW PROC

您可以使用 [SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md) 检查 BE 节点中正在运行的查询数量以及内存和 CPU 使用情况：

```Plain
mysql> SHOW PROC '/backends'\G
*************************** 1. row ***************************
...
    NumRunningQueries: 0
           MemUsedPct: 0.79 %
           CpuUsedPct: 0.0 %
```

### SHOW PROCESSLIST

您可以使用 [SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) 检查查询是否在队列中（当 `IsPending` 为 `true` 时）：

```Plain
mysql> SHOW PROCESSLIST;
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
| Id   | User | Host                | Db    | Command | ConnectionStartTime | Time | State | Info              | IsPending |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
|    2 | root | xxx.xx.xxx.xx:xxxxx |       | Query   | 2022-11-24 18:08:29 |    0 | OK    | SHOW PROCESSLIST  | false     |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
```

### FE 审计日志

您可以检查 FE 审计日志文件 **fe.audit.log**。字段 `PendingTimeMs` 表示查询在队列中等待的时间，单位是毫秒。

### FE 指标

以下 FE 指标来自每个 FE 节点的统计数据。

|**指标**|**单位**|**类型**|**描述**|
|---|---|---|---|
|starrocks_fe_query_queue_pending|计数|瞬时|队列中当前查询的数量。|
|starrocks_fe_query_queue_total|计数|瞬时|历史上排队的查询总数（包括当前正在运行的查询）。|
|starrocks_fe_query_queue_timeout|计数|瞬时|队列中超时的查询总数。|
|starrocks_fe_resource_group_query_queue_total|计数|瞬时|历史上在此资源组中排队的查询总数（包括当前正在运行的查询）。`name` 标签表示资源组的名称。从 v3.1.4 开始支持此指标。|
|starrocks_fe_resource_group_query_queue_pending|计数|瞬时|当前在此资源组的队列中的查询数。`name` 标签表示资源组的名称。从 v3.1.4 开始支持此指标。|
|starrocks_fe_resource_group_query_queue_timeout|计数|瞬时|此资源组队列中已超时的查询数。`name` 标签表示资源组的名称。从 v3.1.4 开始支持此指标。|

### SHOW RUNNING QUERIES

从 v3.1.4 开始，StarRocks 支持 SQL 语句 `SHOW RUNNING QUERIES`，用于显示每个查询的队列信息。各字段含义如下：

- `QueryId`：查询的 ID。
- `ResourceGroupId`：查询命中的资源组 ID。当没有命中用户定义的资源组时，显示为“-”。
- `StartTime`：查询的开始时间。
- `PendingTimeout`：PENDING 查询在队列中的超时时间。
- `QueryTimeout`：查询的超时时间。
- `State`：查询的队列状态，“PENDING”表示它在队列中，“RUNNING”表示它当前正在执行。
- `Slots`：查询请求的逻辑资源数量，目前固定为 `1`。
- `Frontend`：发起查询的 FE 节点。
- `FeStartTime`：发起查询的 FE 节点的开始时间。

示例：

```Plain
MySQL [(none)]> SHOW RUNNING QUERIES;
+--------------------------------------+-----------------+---------------------+---------------------+---------------------+-----------+-------+---------------------------------+---------------------+
| QueryId                              | ResourceGroupId | StartTime           | PendingTimeout      | QueryTimeout        |   State   | Slots | Frontend                        | FeStartTime         |
+--------------------------------------+-----------------+---------------------+---------------------+---------------------+-----------+-------+---------------------------------+---------------------+
| a46f68c6-3b49-11ee-8b43-00163e10863a | -               | 2023-08-15 16:56:37 | 2023-08-15 17:01:37 | 2023-08-15 17:01:37 |  RUNNING  | 1     | 127.00.00.01_9010_1692069711535 | 2023-08-15 16:37:03 |
| a6935989-3b49-11ee-935a-00163e13bca3 | 12003           | 2023-08-15 16:56:40 | 2023-08-15 17:01:40 | 2023-08-15 17:01:40 |  RUNNING  | 1     | 127.00.00.02_9010_1692069658426 | 2023-08-15 16:37:03 |
```
# 查询队列

本主题介绍了如何在StarRocks中管理查询队列。

从v2.5版本开始，StarRocks支持查询队列。启用查询队列后，当并发阈值或资源限制达到时，StarRocks会自动将传入的查询排队，从而避免过载恶化。待处理的查询在队列中等待，直到有足够的计算资源可用来开始执行。从v3.1.4版本开始，StarRocks支持在资源组级别设置查询队列。

您可以设置CPU使用率、内存使用率和查询并发性的阈值来触发查询队列。

**路线图**：

|版本|全局查询队列|资源组级查询队列|集体并发管理|动态并发调整|
|---|---|---|---|---|
|v2.5|✅|❌|❌|❌|
|v3.1.4|✅|✅|✅|✅|

## 启用查询队列

默认情况下，查询队列是禁用的。您可以通过设置相应的全局会话变量，为INSERT加载、SELECT查询和统计查询启用全局或资源组级查询队列。

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

### 启用资源组级查询队列

从v3.1.4版本开始，StarRocks支持在资源组级别设置查询队列。

要启用资源组级查询队列，除了上面提到的全局会话变量，您还需要设置`enable_group_level_query_queue`。

```SQL
SET GLOBAL enable_group_level_query_queue = true;
```

## 指定资源阈值

### 为全局查询队列指定资源阈值

您可以通过以下全局会话变量设置触发查询队列的阈值：

|**变量**|**默认值**|**描述**|
|---|---|---|
|query_queue_concurrency_limit|0|BE上并发查询的上限。只有设置大于`0`后才生效。|
|query_queue_mem_used_pct_limit|0|BE上内存使用百分比的上限。只有设置大于`0`后才生效。范围：[0, 1]|
|query_queue_cpu_used_permille_limit|0|BE上CPU使用千分比的上限（CPU使用 * 1000）。只有设置大于`0`后才生效。范围：[0, 1000]|

> **注意**
> 默认情况下，BE每秒向FE报告一次资源使用情况。您可以通过设置BE配置项`report_resource_usage_interval_ms`来更改此间隔。

### 为资源组级查询队列指定资源阈值

从v3.1.4版本开始，您可以在创建资源组时设置单独的并发限制（`concurrency_limit`）和CPU核心限制（`max_cpu_cores`）。当查询启动时，如果全局或资源组级别的任何资源消耗超过资源阈值，查询将被放入队列，直到所有资源消耗都在阈值之内。

|**变量**|**默认值**|**描述**|
|---|---|---|
|concurrency_limit|0|单个BE节点上资源组的并发限制。只有设置为大于`0`时才生效。|
|max_cpu_cores|0|单个BE节点上此资源组的CPU核心限制。只有设置为大于`0`时才生效。范围：[0, `avg_be_cpu_cores`]，其中`avg_be_cpu_cores`表示所有BE节点上CPU核心的平均数量。|

您可以使用SHOW USAGE RESOURCE GROUPS来查看每个资源组在每个BE节点上的资源使用信息，如[查看资源组使用信息](./resource_group.md#view-resource-group-usage-information)中所述。

### 管理查询并发

当运行中的查询数量（`num_running_queries`）超过全局或资源组的`concurrency_limit`时，传入的查询将被放入队列。获取`num_running_queries`的方式在版本&lt; v3.1.4和&ge; v3.1.4之间有所不同。

- 在版本&lt; v3.1.4中，`num_running_queries`由BEs在`report_resource_usage_interval_ms`指定的间隔内报告。因此，在识别`num_running_queries`的变化时可能会有一些延迟。例如，如果BEs此刻报告的`num_running_queries`没有超过全局或资源组的`concurrency_limit`，但在下一次报告之前到达的传入查询超过了`concurrency_limit`，这些传入查询将不会等待在队列中而直接执行。

- 在版本&ge; v3.1.4中，所有运行中的查询由Leader FE集体管理。每个Follower FE在启动或结束查询时都会通知Leader FE，这使得StarRocks能够处理查询突然增加超过`concurrency_limit`的场景。

## 配置查询队列

您可以通过以下全局会话变量设置查询队列的容量和队列中查询的最大超时时间：

|**变量**|**默认值**|**描述**|
|---|---|---|
|query_queue_max_queued_queries|1024|队列中查询的上限。当达到此阈值时，传入的查询将被拒绝。只有设置大于`0`后才生效。|
|query_queue_pending_timeout_second|300|队列中待处理查询的最大超时时间。达到此阈值时，相应的查询将被拒绝。单位：秒。|

## 配置动态调整查询并发

从v3.1.4版本开始，对于由查询队列管理并由Pipeline Engine运行的查询，StarRocks可以根据当前运行中的查询数量`num_running_queries`、片段数量`num_fragments`和查询并发`pipeline_dop`动态调整传入查询的查询并发`pipeline_dop`。这允许您在最小化调度开销的同时动态控制查询并发，确保BE资源的最佳利用。有关片段和查询并发`pipeline_dop`的更多信息，请参见[查询管理 - 调整查询并发](./Query_management.md)。

对于查询队列下的每个查询，StarRocks都维护了一个驱动程序的概念，它代表了单个BE上查询的并发片段。其逻辑值`num_drivers`，代表了单个BE上该查询所有片段的总并发数，等于`num_fragments * pipeline_dop`。当新查询到达时，StarRocks根据以下规则调整查询并发`pipeline_dop`：

- 运行中的驱动程序数量`num_drivers`超过并发驱动程序的低水位限制`query_queue_driver_low_water`越多，查询并发`pipeline_dop`调整得越低。
- StarRocks限制运行中的驱动程序数量`num_drivers`低于查询的并发驱动程序的高水位限制`query_queue_driver_high_water`。

您可以使用以下全局会话变量配置查询并发`pipeline_dop`的动态调整：

|**变量**|**默认值**|**描述**|
|---|---|---|
|query_queue_driver_high_water|-1|查询的并发驱动程序的高水位限制。只有设置为非负值时才生效。当设置为`0`时，相当于`avg_be_cpu_cores * 16`，其中`avg_be_cpu_cores`表示所有BE节点上CPU核心的平均数量。当设置为大于`0`的值时，直接使用该值。|
|query_queue_driver_low_water|-1|查询的并发驱动程序的低水位限制。只有设置为非负值时才生效。当设置为`0`时，相当于`avg_be_cpu_cores * 8`。当设置为大于`0`的值时，直接使用该值。|

## 监控查询队列

您可以使用以下方法查看与查询队列相关的信息。

### SHOW PROC

您可以使用[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md)检查BE节点中的运行查询数量以及内存和CPU使用情况：

```Plain
mysql> SHOW PROC '/backends'\G
*************************** 1. row ***************************
...
    NumRunningQueries: 0
           MemUsedPct: 0.79 %
           CpuUsedPct: 0.0 %
```

### SHOW PROCESSLIST

您可以使用[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md)检查查询是否在队列中（当`IsPending`为`true`时）：

```Plain
mysql> SHOW PROCESSLIST;
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
| Id   | User | Host                | Db    | Command | ConnectionStartTime | Time | State | Info              | IsPending |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
|    2 | root | xxx.xx.xxx.xx:xxxxx |       | Query   | 2022-11-24 18:08:29 |    0 | OK    | SHOW PROCESSLIST  | false     |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
```

### FE审计日志

您可以检查FE审计日志文件**fe.audit.log**。字段`PendingTimeMs`表示查询在队列中等待的时间，其单位是毫秒。

### FE指标

以下FE指标来自每个FE节点的统计数据。

|指标|单位|类型|描述|
|---|---|---|---|
|starrocks_fe_query_queue_pending|计数|瞬时|队列中当前的查询数量。|
|starrocks_fe_query_queue_total|计数|瞬时|历史上队列中的查询总数（包括当前运行的）。|
|starrocks_fe_query_queue_timeout|计数|瞬时|在队列中超时的查询总数。|
|starrocks_fe_resource_group_query_queue_total|计数|瞬时|此资源组历史上队列中的查询总数（包括当前运行的）。`name`标签表示资源组的名称。此指标从v3.1.4版本开始支持。|
|starrocks_fe_resource_group_query_queue_pending|计数|瞬时|当前此资源组队列中的查询数量。`name`标签表示资源组的名称。此指标从v3.1.4版本开始支持。|
|starrocks_fe_resource_group_query_queue_timeout|计数|瞬时|此资源组队列中超时的查询数量。`name`标签表示资源组的名称。此指标从v3.1.4版本开始支持。|

### SHOW RUNNING QUERIES

从v3.1.4版本开始，StarRocks支持SQL语句`SHOW RUNNING QUERIES`，用于显示每个查询的队列信息。每个字段的含义如下：

- `QueryId`：查询的ID。
- `ResourceGroupId`：查询命中的资源组ID。当没有命中用户定义的资源组时，将显示为"-"。
- `StartTime`：查询的开始时间。
- `PendingTimeout`：PENDING查询在队列中的超时时间。
- `QueryTimeout`：查询的超时时间。
- `State`：查询的队列状态，其中"PENDING"表示它在队列中，"RUNNING"表示它当前正在执行。
- `Slots`：查询请求的逻辑资源量，当前固定为`1`。
- `Frontend`：发起查询的FE节点。
- `FeStartTime`：发起查询的FE节点的开始时间。

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