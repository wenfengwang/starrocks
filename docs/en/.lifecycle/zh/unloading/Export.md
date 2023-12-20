---
displayed_sidebar: English
---

# 使用 EXPORT 导出数据

本主题介绍如何从您的 StarRocks 集群中指定的表或分区导出数据，将其作为 CSV 数据文件导出到外部存储系统，该存储系统可以是分布式文件系统 HDFS 或云存储系统，例如 AWS S3。

> **注意**
> 您只能作为具有对那些 StarRocks 表 **EXPORT** 权限的用户，才能从 StarRocks 表中导出数据。如果您没有 **EXPORT** 权限，请按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中提供的指导来授予您用于连接 StarRocks 集群的用户 **EXPORT** 权限。

## 背景信息

在 v2.4 及更早版本中，StarRocks 依赖代理来建立 StarRocks 集群与外部存储系统之间的连接，以使用 EXPORT 语句导出数据。因此，您需要在 EXPORT 语句中输入 `WITH BROKER "<broker_name>"` 来指定您想使用的代理。这称为“基于代理的卸载”。代理是一个独立的、无状态的服务，它与文件系统接口集成，帮助 StarRocks 将数据导出到您的外部存储系统。

从 v2.5 开始，StarRocks 使用 EXPORT 语句导出数据时不再依赖代理来建立与外部存储系统之间的连接。因此，您不再需要在 EXPORT 语句中指定代理，但您仍需保留 `WITH BROKER` 关键词。这称为“无代理卸载”。

然而，当您的数据存储在 HDFS 中时，无代理卸载可能不适用，您可以使用基于代理的卸载：

- 如果您要将数据导出到多个 HDFS 集群，您需要为这些 HDFS 集群中的每一个部署和配置一个独立的代理。
- 如果您要将数据导出到单个 HDFS 集群，并且您已配置多个 Kerberos 用户，您需要部署一个独立的代理。

> **注意**
> 您可以使用 [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) 语句来检查您的 StarRocks 集群中部署的代理。如果没有部署代理，您可以按照 [部署代理](../deployment/deploy_broker.md) 中提供的指导来部署代理。

## 支持的存储系统

- 分布式文件系统 HDFS
- 例如 AWS S3 的云存储系统

## 注意事项

- 我们建议您一次导出的数据量不要超过几十 GB。如果您一次导出的数据量过大，可能会导致导出失败，并且重试导出的成本会增加。

- 如果源 StarRocks 表包含大量数据，我们建议您每次只从表的几个分区导出数据，直到表中的所有数据都被导出。

- 如果在导出作业运行时，StarRocks 集群中的 FE 重新启动或选举出新的领导 FE，导出作业会失败。在这种情况下，您必须重新提交导出作业。

- 如果在导出作业完成后，StarRocks 集群中的 FE 重新启动或选举出新的领导 FE，[SHOW EXPORT](../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md) 语句返回的作业信息可能会丢失。

- StarRocks 仅导出基础表的数据，不会导出在基础表上创建的物化视图的数据。

- 导出作业需要进行数据扫描，这会占用 I/O 资源，从而增加查询延迟。

## 工作流程

提交导出作业后，StarRocks 会识别出导出作业涉及的所有平板。然后，StarRocks 将涉及的平板分组并生成查询计划。查询计划用于从涉及的平板中读取数据，并将数据写入目标存储系统的指定路径。

下图显示了一般工作流程。

![img](../assets/5.3.1-1.png)

一般工作流程包括以下三个步骤：

1. 用户向领导 FE 提交导出作业。

2. 领导 FE 向 StarRocks 集群中的所有 BE 发出 `snapshot` 指令，以便 BE 可以对涉及的平板进行快照，确保导出数据的一致性。领导 FE 还生成多个导出任务。每个导出任务都是一个查询计划，每个查询计划用于处理涉及的平板的一部分。

3. 领导 FE 将导出任务分配给 BE。

## 原则

当 StarRocks 执行查询计划时，它首先在目标存储系统的指定路径中创建一个名为 `__starrocks_export_tmp_xxx` 的临时文件夹。在临时文件夹的名称中，`xxx` 表示导出作业的 ID。一个临时文件夹的示例名称是 `__starrocks_export_tmp_921d8f80-7c9d-11eb-9342-acde48001122`。StarRocks 成功执行一个查询计划后，会在临时文件夹中生成一个临时文件，并将导出的数据写入该临时文件。

所有数据导出后，StarRocks 使用 RENAME 语句将生成的临时文件保存到指定路径。

## 相关参数

本节描述了一些与导出相关的参数，您可以在 StarRocks 集群的 FE 中配置这些参数。

- `export_checker_interval_second`：导出作业调度的时间间隔。默认间隔是 5 秒。在 FE 中重新配置此参数后，需要重启 FE 以使新的参数设置生效。

- `export_running_job_num_limit`：允许的正在运行的导出作业的最大数量。如果正在运行的导出作业数量超过此限制，超出的导出作业在执行 `snapshot` 后会进入等待状态。默认最大数量是 5。您可以在导出作业运行时重新配置此参数。

- `export_task_default_timeout_second`：导出作业的超时时间。默认超时时间是 2 小时。您可以在导出作业运行时重新配置此参数。

- `export_max_bytes_per_be_per_task`：每个 BE 的每个导出任务可以导出的最大压缩数据量。此参数提供了一个策略，基于此 StarRocks 将导出作业分割为可以并发运行的导出任务。默认最大量是 256 MB。

- `export_task_pool_size`：线程池可以并发运行的导出任务的最大数量。默认最大数量是 5。

## 基本操作

### 提交导出作业

假设您的 StarRocks 数据库 `db1` 包含一个名为 `tbl1` 的表。要将 `tbl1` 的分区 `p1` 和 `p2` 中的列 `col1` 和 `col3` 的数据导出到您的 HDFS 集群的 `export` 路径，请运行以下命令：

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

有关详细的语法和参数描述以及导出数据到 AWS S3 的命令示例，请参阅 [EXPORT](../sql-reference/sql-statements/data-manipulation/EXPORT.md)。

### 获取导出作业的查询 ID

提交导出作业后，您可以使用 SELECT LAST_QUERY_ID() 语句查询导出作业的查询 ID。有了查询 ID，您就可以查看或取消导出作业。

详细的语法和参数描述，请参阅 [last_query_id](../sql-reference/sql-functions/utility-functions/last_query_id.md)。

### 查看导出作业的状态

提交导出作业后，您可以使用 SHOW EXPORT 语句来查看导出作业的状态。例如：

```SQL
SHOW EXPORT WHERE queryid = "edee47f0-abe1-11ec-b9d1-00163e1e238f";
```

> **注意**
> 在上述示例中，`queryid` 是导出作业的查询 ID。

返回的信息类似于以下输出：

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

详细的语法和参数描述，请参阅 [SHOW EXPORT](../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md)。

### 取消导出作业

您可以使用 CANCEL EXPORT 语句来取消您已提交的导出作业。例如：

```SQL
CANCEL EXPORT WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";
```

> **注意**
> 在上述示例中，`queryid` 是导出作业的查询 ID。

详细的语法和参数描述，请参阅 [CANCEL EXPORT](../sql-reference/sql-statements/data-manipulation/CANCEL_EXPORT.md)。

## 最佳实践

### 查询计划拆分

导出作业分割成的查询计划数量取决于导出作业涉及的平板数量以及每个查询计划可处理的最大数据量。导出作业作为查询计划进行重试。如果查询计划处理的数据量超过允许的最大量，则查询计划可能会遇到远程存储的抖动等错误。因此，重试查询计划的成本会增加。每个 BE 的每个查询计划可以处理的最大数据量由 `export_max_bytes_per_be_per_task` 参数指定，其默认值为 256 MB。在查询计划中，每个 BE 至少分配一个平板，并且可以导出的数据量不超过 `export_max_bytes_per_be_per_task` 参数指定的限制。

一个导出作业的多个查询计划是并行执行的。您可以使用 FE 参数 `export_task_pool_size` 来指定线程池允许并行运行的导出任务的最大数量。该参数默认值为 `5`。

在正常情况下，导出作业的每个查询计划仅包含扫描和导出两部分。执行查询计划所需的计算逻辑不会消耗太多内存。因此，默认的 2 GB 内存限制可以满足您的大部分业务需求。但在某些情况下，例如当查询计划需要在一个 BE 上扫描多个平板或一个平板有多个版本时，2 GB 的内存容量可能不足。在这些情况下，您需要使用 `load_mem_limit` 参数来指定更高的内存容量限制，例如 4 GB 或 8 GB。