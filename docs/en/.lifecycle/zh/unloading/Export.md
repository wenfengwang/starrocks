---
displayed_sidebar: English
---

# 使用 EXPORT 导出数据

本主题介绍如何将 StarRocks 集群中指定的表或分区的数据以 CSV 数据文件的形式导出到外部存储系统，可以是分布式文件系统 HDFS，也可以是云存储系统，比如 AWS S3。

> **注意**
>
> 您只能以对 StarRocks 表具有 EXPORT 权限的用户身份从 StarRocks 表中导出数据。如果您没有 EXPORT 权限，请按照[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)中提供的说明，将 EXPORT 权限授予您用于连接到 StarRocks 集群的用户。

## 背景信息

在 v2.4 及更早版本中，当使用 EXPORT 语句导出数据时，StarRocks 集群和外部存储系统之间需要依赖代理来建立连接。因此，在 EXPORT 语句中需要输入 `WITH BROKER "<broker_name>"` 来指定要使用的代理。这称为“基于代理的卸载”。代理是一种独立的无状态服务，集成了文件系统接口，帮助 StarRocks 将数据导出到外部存储系统。

从 v2.5 开始，当使用 EXPORT 语句导出数据时，StarRocks 不再依赖代理来建立 StarRocks 集群和外部存储系统的连接。因此，您不再需要在 EXPORT 语句中指定代理，但仍需要保留 `WITH BROKER` 关键字。这称为“无代理卸载”。

但是，当您的数据存储在 HDFS 中时，无代理卸载可能不起作用，您可以使用基于代理的卸载：

- 如果要将数据导出到多个 HDFS 集群，则需要为每个 HDFS 集群部署和配置一个独立的代理。
- 如果要将数据导出到单个 HDFS 集群，并且配置了多个 Kerberos 用户，则需要部署一个独立的代理。

> **注意**
>
> 您可以使用 [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) 语句来查看在 StarRocks 集群中部署的代理。如果未部署代理，您可以按照[部署代理](../deployment/deploy_broker.md)中的说明来部署代理。

## 支持的存储系统

- 分布式文件系统 HDFS
- 云存储系统，比如 AWS S3

## 注意事项

- 我们建议一次导出的数据量不要超过几十GB。如果一次导出的数据量过大，导出可能会失败，并且重试导出的成本会增加。

- 如果源 StarRocks 表包含大量数据，建议您每次只从表的几个分区导出数据，直到导出表的所有数据为止。

- 如果 StarRocks 集群中的 FE 重新启动，或者在运行导出作业时选择了新的 leader FE，则导出作业会失败。在这种情况下，您必须重新提交导出作业。

- 导出作业完成后，如果 StarRocks 集群中的 FE 重新启动或选择了新的 leader FE，则[SHOW EXPORT](../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md)语句返回的部分作业信息可能会丢失。

- StarRocks 仅导出基表的数据。StarRocks 不会导出在基表上创建的物化视图的数据。

- 导出作业需要进行数据扫描，这会占用 I/O 资源，从而增加查询延迟。

## 工作流程

提交导出作业后，StarRocks 会识别导出作业中涉及的所有 Tablet。然后，StarRocks 将涉及的 Tablet 分组并生成查询计划。查询计划用于从涉及的 Tablet 读取数据，并将数据写入目标存储系统的指定路径。

下图显示了常规工作流程。

![图片](../assets/5.3.1-1.png)

常规工作流程包括以下三个步骤：

1. 用户向 leader FE 提交导出作业。

2. leader FE 向 StarRocks 集群中的所有 BE 发送 `snapshot` 指令，以便 BE 对涉及的 Tablet 进行快照，以保证导出数据的一致性。leader FE 还会生成多个导出任务。每个导出任务都是一个查询计划，每个查询计划用于处理涉及的 Tablet 的一部分数据。

3. leader FE 将导出任务分发给 BE。

## 原则

当 StarRocks 执行查询计划时，首先会在目标存储系统的指定路径下创建一个名为 `__starrocks_export_tmp_xxx` 的临时文件夹。在临时文件夹的名称中，`xxx` 表示导出作业的 ID。临时文件夹的示例名称是 `__starrocks_export_tmp_921d8f80-7c9d-11eb-9342-acde48001122`。StarRocks 成功执行查询计划后，会在临时文件夹中生成一个临时文件，并将导出的数据写入生成的临时文件中。

导出所有数据后，StarRocks 使用 RENAME 语句将生成的临时文件保存到指定路径。

## 相关参数

本节描述了您可以在 StarRocks 集群的 FE 中配置的一些与导出相关的参数。

- `export_checker_interval_second`：安排导出作业的时间间隔。默认间隔为 5 秒。在重新配置此参数后，需要重新启动 FE 才能使新的参数设置生效。

- `export_running_job_num_limit`：允许运行的导出作业的最大数量。如果正在运行的导出作业数超过此限制，过多的导出作业将在运行 `snapshot` 后进入等待状态。默认的最大数量为 5。您可以在运行导出作业时重新配置此参数。

- `export_task_default_timeout_second`：导出作业的超时时间。默认超时期限为 2 小时。您可以在运行导出作业时重新配置此参数。

- `export_max_bytes_per_be_per_task`：每个导出任务可以从每个 BE 导出的最大压缩数据量。该参数提供了一个策略，StarRocks 根据该策略将导出作业拆分为可以并发运行的导出任务。默认最大大小为 256 MB。

- `export_task_pool_size`：线程池可以并发运行的最大导出任务数。默认的最大数量为 5。

## 基本操作

### 提交导出作业

假设您的 StarRocks 数据库 `db1` 包含一个名为 `tbl1` 的表。要将 `tbl1` 的分区 `p1` 和 `p2` 中的 `col1` 和 `col3` 列的数据导出到 HDFS 集群的 `export` 路径，请运行以下命令：

```SQL
EXPORT TABLE db1.tbl1 
PARTITION (p1,p2)
(col1, col3)
TO "hdfs://HDFS_IP:HDFS_Port/export/lineorder_" 
PROPERTIES
(
    "column_separator"=",",
    "load_mem_limit"="2147483648",
    "timeout" = "3600"
)
WITH BROKER
(
    "username" = "user",
    "password" = "passwd"
);
```

有关详细的语法和参数说明，以及将数据导出到 AWS S3 的命令示例，请参阅 [EXPORT](../sql-reference/sql-statements/data-manipulation/EXPORT.md)。

### 获取导出作业的查询 ID

提交导出作业后，您可以使用 SELECT LAST_QUERY_ID() 语句查询导出作业的查询 ID。有了查询 ID，您可以查看或取消导出作业。

有关详细的语法和参数说明，请参阅 [last_query_id](../sql-reference/sql-functions/utility-functions/last_query_id.md)。

### 查看导出作业的状态

提交导出作业后，您可以使用 SHOW EXPORT 语句查看导出作业的状态。例如：

```SQL
SHOW EXPORT WHERE queryid = "edee47f0-abe1-11ec-b9d1-00163e1e238f";
```

> **注意**
>
> 在上面的示例中，`queryid` 是导出作业的查询 ID。

将返回类似以下输出的信息：

```Plain
JobId: 14008
State: FINISHED
Progress: 100%
TaskInfo: {"partitions":["*"],"mem limit":2147483648,"column separator":",","line delimiter":"\n","tablet num":1,"broker":"hdfs","coord num":1,"db":"default_cluster:db1","tbl":"tbl3",columns:["col1", "col3"]}
Path: oss://bj-test/export/
CreateTime: 2019-06-25 17:08:24
StartTime: 2019-06-25 17:08:28
FinishTime: 2019-06-25 17:08:34
Timeout: 3600
ErrorMsg: N/A
```

有关详细的语法和参数说明，请参阅 [SHOW EXPORT](../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md)。

### 取消导出作业

您可以使用 CANCEL EXPORT 语句取消您提交的导出作业。例如：

```SQL
CANCEL EXPORT WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```

> **注意**
>
> 在上面的示例中，`queryid` 是导出作业的查询 ID。

有关详细的语法和参数说明，请参阅 [CANCEL EXPORT](../sql-reference/sql-statements/data-manipulation/CANCEL_EXPORT.md)。

## 最佳实践

### 查询计划拆分

导出作业被拆分为多个查询计划的数量取决于导出作业涉及的 Tablet 数量以及每个查询计划可以处理的最大数据量。导出作业会作为查询计划进行重试。如果查询计划处理的数据量超过允许的最大数据量，查询计划会遇到远程存储抖动等错误。因此，重试查询计划的成本会增加。每个 BE 每个查询计划可以处理的最大数据量由参数 `export_max_bytes_per_be_per_task` 指定，默认为 256 MB。在查询计划中，每个 BE 至少分配一个 Tablet，并且可以导出不超过参数 `export_max_bytes_per_be_per_task` 指定限制的数据量。

导出作业的多个查询计划是并发执行的。您可以使用 FE 参数 `export_task_pool_size` 指定线程池允许并发运行的最大导出任务数，默认为 `5`。

一般情况下，导出作业的每个查询计划只由扫描和导出两部分组成。执行查询计划所需的计算的逻辑不会占用太多内存。因此，默认的内存限制为 2 GB 可以满足大部分业务需求。但是，在某些情况下，例如当查询计划需要在 BE 上扫描多个 Tablet 或 Tablet 具有多个版本时，2 GB 的内存容量可能不足。在这些情况下，您需要使用 `load_mem_limit` 参数指定更高的内存容量限制，比如 4 GB 或 8 GB。
