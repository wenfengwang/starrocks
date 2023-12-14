---
displayed_sidebar: "Chinese"
---

# 使用EXPORT导出数据

本主题介绍如何将StarRocks集群中指定表或分区的数据导出为CSV数据文件到外部存储系统。这些系统可以是分布式文件系统HDFS，也可以是云存储系统，例如AWS S3。

> **注意**
>
> 只有在StarRocks表上具有EXPORT权限的用户才能导出StarRocks表中的数据。如果您没有EXPORT权限，请按照[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)中提供的说明，为连接到StarRocks集群的用户授予EXPORT权限。

## 背景信息

在v2.4及更早版本中，StarRocks在使用EXPORT语句导出数据时依赖于代理程序来建立StarRocks集群与外部存储系统之间的连接，因此您需要在EXPORT语句中输入`WITH BROKER "<broker_name>"`以指定要在EXPORT语句中使用的代理程序。这称为"基于代理程序的卸载"。代理程序是一个独立的、无状态的服务，集成了文件系统接口，帮助StarRocks将数据导出到您的外部存储系统。

从v2.5开始，StarRocks在使用EXPORT语句导出数据时不再依赖于代理程序来建立StarRocks集群与外部存储系统之间的连接。因此，您不再需要在EXPORT语句中指定代理程序，但仍需要保留`WITH BROKER`关键字。这称为"无代理程序的卸载"。

然而，当您的数据存储在HDFS中时，无代理程序的卸载可能无法正常工作，您可以转而使用基于代理程序的卸载：

- 如果您要将数据导出到多个HDFS集群，您需要为这些HDFS集群部署和配置独立的代理程序。
- 如果您要将数据导出到单个HDFS集群，并且已经配置了多个Kerberos用户，则需要部署一个独立的代理程序。

> **注意**
>
> 您可以使用[SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md)语句来检查在您的StarRocks集群中部署的代理程序。如果没有部署代理程序，您可以按照[部署代理程序](../deployment/deploy_broker.md)中提供的说明来部署代理程序。

## 支持的存储系统

- 分布式文件系统HDFS
- 云存储系统，例如AWS S3

## 注意事项

- 我们建议每次导出的数据不要超过几十GB。如果一次导出了过多数据，可能会导致导出失败并增加重试导出的成本。

- 如果源StarRocks表包含大量数据，建议您每次仅从表的少数分区导出数据，直到导出了表的所有数据。

- 如果您的StarRocks集群中的前端FE重启，或在运行导出作业时选举了新的主FE，则导出作业将失败。在这种情况下，您需要重新提交导出作业。

- 如果在导出作业完成后，您的StarRocks集群中的前端FE重启，或选举了新的主FE，则[SHOW EXPORT](../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md)语句返回的部分作业信息可能会丢失。

- StarRocks仅导出基本表的数据，不会导出基本表上创建的物化视图的数据。

- 导出作业需要数据扫描，占用I/O资源，从而增加查询延迟。

## 工作流程

在您提交一个导出作业后，StarRocks会识别所有参与导出作业的tablet。然后，StarRocks将涉及的tablet分组并生成查询计划。这些查询计划用于从涉及的tablet读取数据，并将数据写入目的存储系统的指定路径。

下图展示了总体工作流程。

![img](../assets/5.3.1-1.png)

总体工作流程包括以下三个步骤：

1. 用户向主FE提交一个导出作业。

2. 主FE向StarRocks集群中的所有BE发出`snapshot`指令，使BE可以对涉及的tablet进行快照，以确保要导出的数据的一致性。主FE还生成多个导出任务。每个导出任务都是一个查询计划，每个查询计划用于处理涉及的tablet的一部分。

3. 主FE将导出任务分发给BE。

## 原则

在StarRocks执行查询计划时，它首先在目的存储系统的指定路径中创建一个名为`__starrocks_export_tmp_xxx`的临时文件夹。在临时文件夹的名称中，`xxx`代表导出作业的ID。临时文件夹的一个示例名称是`__starrocks_export_tmp_921d8f80-7c9d-11eb-9342-acde48001122`。在StarRocks成功执行一个查询计划后，它会在临时文件夹中生成一个临时文件，并将导出的数据写入生成的临时文件中。

在导出所有数据后，StarRocks使用RENAME语句将生成的临时文件保存到指定路径。

## 相关参数

本节描述了您可以在StarRocks集群的FE中配置的一些与导出相关的参数。

- `export_checker_interval_second`: 调度导出作业的间隔。默认间隔为5秒。在为FE重新配置此参数后，需要重新启动FE以使新参数设置生效。

- `export_running_job_num_limit`: 允许的最大运行导出作业数。如果运行的导出作业数量超过此限制，则过多的导出作业在运行`snapshot`后进入等待状态。默认最大数量为5。您可以在运行导出作业时重新配置此参数。

- `export_task_default_timeout_second`: 导出作业的超时时间。默认超时时间为2小时。您可以在运行导出作业时重新配置此参数。

- `export_max_bytes_per_be_per_task`: 每个BE从每个导出任务中最多可以导出的数据量（压缩后）。该参数提供了StarRocks将导出作业拆分为可以并发运行的导出任务的策略。默认最大数量为256 MB。

- `export_task_pool_size`: 线程池可以并发运行的最大导出任务数。默认最大数量为5。

## 基本操作

### 提交导出作业

假设您的StarRocks数据库`db1`包含一个名为`tbl1`的表。要将`tbl1`的分区`p1`和`p2`中`col1`和`col3`的数据导出到HDFS集群的`export`路径，运行以下命令：

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

有关详细的语法和参数说明，以及导出数据到AWS S3的命令示例，请参见[EXPORT](../sql-reference/sql-statements/data-manipulation/EXPORT.md)。

### 获取导出作业的查询ID

提交导出作业后，您可以使用SELECT LAST_QUERY_ID()语句查询导出作业的查询ID。有了查询ID，您可以查看或取消导出作业。

有关详细的语法和参数说明，请参见[last_query_id](../sql-reference/sql-functions/utility-functions/last_query_id.md)。

### 查看导出作业的状态

提交导出作业后，您可以使用SHOW EXPORT语句查看导出作业的状态。例如：

```SQL
SHOW EXPORT WHERE queryid = "edee47f0-abe1-11ec-b9d1-00163e1e238f";
```

返回类似以下输出的信息：

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

有关详细的语法和参数说明，请参见[SHOW EXPORT](../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md)。

### 取消导出作业

您可以使用CANCEL EXPORT语句来取消您提交的导出作业。例如：

```SQL
取消导出 WHERE queryid = "921d8f80-7c9d-11eb-9342-acde48001122";


> **说明**
>
> 在上述示例中，`queryid` 是导出作业的查询 ID。

有关详细语法和参数说明，请参见 [CANCEL EXPORT](../sql-reference/sql-statements/data-manipulation/CANCEL_EXPORT.md)。

## 最佳实践

### 查询计划拆分

导出作业分为多个查询计划，分为数取决于参与导出作业的 Tablet 数量以及每个查询计划可处理的最大数据量。导出作业会作为查询计划进行重试。如果查询计划处理的数据量超过了允许的最大量，查询计划将遇到远程存储中的抖动等错误。因此，重试查询计划的成本将增加。每个 BE 可以处理的最大数据量由 `export_max_bytes_per_be_per_task` 参数指定，默认为 256 MB。在查询计划中，每个 BE 至少分配一个 Tablet，并且可以导出的数据量不超过 `export_max_bytes_per_be_per_task` 参数指定的限制。

导出作业的多个查询计划会并发执行。您可以使用 FE 参数 `export_task_pool_size` 来指定允许线程池并发运行的最大导出任务数。该参数默认为 `5`。

在正常情况下，导出作业的每个查询计划仅包括扫描和导出两个部分，执行查询计划所需的计算逻辑不会消耗过多内存。因此，默认的 2 GB 内存限制可以满足大多数业务需求。然而，在某些情况下，比如当查询计划需要扫描一个 BE 上的许多 Tablets，或者一个 Tablet 具有许多版本时，默认的 2 GB 内存容量可能不足。在这些情况下，您需要使用 `load_mem_limit` 参数来指定更高的内存容量限制，例如 4 GB 或 8 GB。