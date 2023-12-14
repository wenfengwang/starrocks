---
displayed_sidebar: "Chinese"
---

# 数据加载概述

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

数据加载是根据业务需求从各种数据源中清洗和转换原始数据，并将生成的数据加载到StarRocks中，以便进行高速数据分析。

您可以通过运行加载作业将数据加载到StarRocks。每个加载作业都有一个用户指定或由StarRocks自动生成的唯一标签来标识。每个标签只能用于一个加载作业。加载作业完成后，其标签不能用于任何其他加载作业。只能重用失败的加载作业的标签。此机制有助于确保与特定标签关联的数据只能加载一次，从而实现至多一次语义。

StarRocks提供的所有加载方式都可以保证原子性。原子性意味着加载作业中的合格数据必须全部成功加载，或者没有一个合格数据成功加载。决不会出现一些合格数据已经加载而其他数据没有加载的情况。请注意，合格数据不包括由于质量问题（例如数据类型转换错误）而被过滤的数据。

StarRocks支持两种通信协议，可用于提交加载作业：MySQL和HTTP。有关每种加载方式支持的协议的更多信息，请参见本主题的 [加载方式](../loading/Loading_intro.md#loading-methods) 部分。

<InsertPrivNote />

## 支持的数据类型

StarRocks支持加载所有数据类型的数据。您只需注意加载少量特定数据类型的限制。有关更多信息，请参见[数据类型](../sql-reference/sql-statements/data-types/BIGINT.md)。

## 加载模式

StarRocks支持两种加载模式：同步加载模式和异步加载模式。

> **注意**
>
> 如果您使用外部程序加载数据，您必须在决定所选择的加载方式之前选择最适合业务需求的加载模式。

### 同步加载

在同步加载模式下，您提交加载作业后，StarRocks会同步运行作业以加载数据，并在作业完成后返回作业结果。您可以根据作业结果检查作业是否成功。

StarRocks提供两种支持同步加载的加载方式：[Stream Load](../loading/StreamLoad.md) 和 [INSERT](../loading/InsertInto.md)。

同步加载的过程如下：

1. 创建加载作业。

2. 查看StarRocks返回的作业结果。

3. 根据作业结果检查作业是否成功。如果作业结果表示加载失败，您可以重试作业。

### 异步加载

在异步加载模式下，您提交加载作业后，StarRocks会立即返回作业创建结果。

- 如果结果表示作业创建成功，StarRocks会异步运行作业。但这并不意味着数据已成功加载。您必须使用语句或命令来检查作业状态。然后，您可以根据作业状态确定数据是否成功加载。

- 如果结果表示作业创建失败，您可以根据失败信息确定是否需要重试作业。

StarRocks提供三种支持异步加载的加载方式：[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 和 [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

异步加载的过程如下：

1. 创建加载作业。

2. 查看StarRocks返回的作业创建结果，并确定作业是否成功创建。
   a. 如果作业创建成功，请转到步骤 3。
   b. 如果作业创建失败，请返回到步骤 1。

3. 使用语句或命令来检查作业状态，直到作业状态显示 **FINISHED** 或 **CANCELLED**。

Broker加载或Spark加载作业的工作流程包括以下五个阶段，如下图所示。

![Broker Load or Spark Load overflow](../assets/4.1-1.png)

工作流程描述如下：

1. **PENDING**

   作业在队列中等待由FE调度。

2. **ETL**

   FE预处理数据，包括清洗、分区、排序和聚合。
   > **注意**
   >
   > 如果作业是Broker加载作业，则此阶段直接完成。

3. **LOADING**

   FE清理和转换数据，然后发送数据给BE。在加载所有数据后，数据在队列中等待生效。此时，作业状态保持为 **LOADING**。

4. **FINISHED**

   当所有数据生效时，作业状态变为 **FINISHED**。此时，可以查询数据。 **FINISHED** 是最终作业状态。

5. **CANCELLED**

   在作业状态变为 **FINISHED** 之前，您可以随时取消作业。另外，StarRocks可以在加载出错的情况下自动取消作业。作业取消后，作业状态变为 **CANCELLED**。 **CANCELLED** 也是最终作业状态。

Routine作业的工作流程描述如下：

1. 用户从MySQL客户端将作业提交给FE。

2. FE将作业拆分为多个任务。每个任务都被设计为从多个分区加载数据。

3. FE将任务分发给指定的BE。

4. BE执行任务，并在完成任务后向FE报告。

5. 根据BE的报告，FE生成后续任务，重试失败任务（如果有的话），或者根据任务报告挂起任务调度。

## 加载方式

StarRocks提供五种加载方式，可帮助您在各种业务场景中加载数据：[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 和 [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

| 加载方式            | 数据源                                                | 业务场景                                                   | 每个加载作业的数据量                                         | 数据文件格式                                | 加载模式     | 协议    |
| ------------------ | -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------- | ------------ | -------- |
| Stream Load        |  <ul><li>本地文件</li><li>数据流</li></ul> | 通过程序从本地文件系统加载数据文件，或者通过数据流加载数据。  | 10 GB或更少                             |<ul><li>CSV</li><li>JSON</li></ul>               | 同步        | HTTP     |
| Broker Load        | <ul><li>HDFS</li><li>Amazon S3</li><li>Google GCS</li><li>Microsoft Azure Storage</li><li>Alibaba Cloud OSS</li><li>Tencent Cloud COS</li><li>Huawei Cloud OBS</li><li>Other S3-compatible storage system (such as MinIO)</li></ul>| 从HDFS或云存储加载数据。                                              | 几十GB至几百GB                               | <ul><li>CSV</li><li>Parquet</li><li>ORC</li></ul>| 异步        | MySQL    |
| Routine Load       | Apache Kafka®                                       | 实时从Kafka加载数据。                                              | MB至GB级的数据作为小批量                           |<ul><li>CSV</li><li>JSON</li><li>Avro（v3.0.1及以后版本支持）</li></ul>          | 异步        | MySQL    |
| Spark Load         | <ul><li>HDFS</li><li>Hive</li></ul>     |<ul><li>从HDFS或Hive迁移大量数据，使用Apache Spark™集群。</li><li>在使用全局数据字典进行去重的同时加载数据。</li></ul>| 几十GB至TB级                                         |<ul><li>CSV</li><li>ORC（v2.0及以后版本支持）</li><li>Parquet（v2.0及以后版本支持）</ul>       | 异步        | MySQL    |
| INSERT INTO SELECT | <ul><li>StarRocks表</li><li>外部表</li><li>AWS S3</li></ul>**注意**<br />当从AWS S3加载数据时，仅支持Parquet格式或ORC格式文件。     |<ul><li>从外部表加载数据。</li><li>在StarRocks表之间加载数据。</li></ul>| 不固定（数据量根据内存大小而变化。） | StarRocks表      | 同步        | MySQL    |
| INSERT INTO VALUES | <ul><li>程序</li><li>ETL工具</li></ul>    |<ul><li>将少量数据作为单独记录插入。</li><li>使用JDBC等API加载数据。</li></ul>| 少量数据       | SQL       | 同步     | MySQL  |

根据您的业务场景、数据量、数据源、数据文件格式以及加载频率，您可以确定所需的加载方法。此外，在选择加载方法时，请注意以下几点：

- 从Kafka加载数据时，建议使用[常规加载](../loading/RoutineLoad.md)。但是，如果数据需要进行多表连接和提取、转换和加载（ETL）操作，您可以使用Apache Flink®从Kafka读取并预处理数据，然后使用[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)将数据加载到StarRocks中。

- 从Hive、Iceberg、Hudi或Delta Lake加载数据时，建议创建[Hive目录](../data_source/catalog/hive_catalog.md)、[Iceberg目录](../data_source/catalog/iceberg_catalog.md)、[Hudi目录](../data_source/catalog/hudi_catalog.md)或[Delta Lake目录](../data_source/catalog/deltalake_catalog.md)，然后使用[INSERT](../loading/InsertInto.md)加载数据

- 从另一个StarRocks集群或Elasticsearch集群加载数据时，建议创建[StarRocks外部表](../data_source/External_table.md#starrocks-external-table)或[Elasticsearch外部表](../data_source/External_table.md#deprecated-elasticsearch-external-table)，然后使用[INSERT](../loading/InsertInto.md)加载数据。

  > **注意**
  >
  > StarRocks外部表仅支持数据写入，不支持数据读取。

- 从MySQL数据库加载数据时，建议创建[MySQL外部表](../data_source/External_table.md#deprecated-mysql-external-table)，然后使用[INSERT](../loading/InsertInto.md)加载数据。如果要实时加载数据，建议按照[从MySQL进行实时同步](../loading/Flink_cdc_load.md)中提供的说明来加载数据。

- 从其他数据源（如Oracle、PostgreSQL和SQL Server）加载数据时，建议创建[JDBC外部表](../data_source/External_table.md#external-table-for-a-jdbc-compatible-database)，然后使用[INSERT](../loading/InsertInto.md)加载数据。

下图概述了StarRocks支持的各种数据源以及您可以使用的加载数据的方法。

![数据加载来源](../assets/4.1-3.png)

## 内存限制

StarRocks提供了参数，供您限制每个加载作业的内存使用量，从而减少内存消耗，特别是在高并发场景下。但是，不要指定过低的内存使用限制。如果内存使用限制过低，由于加载作业的内存使用达到指定的限制，数据可能会经常从内存刷新到磁盘。建议根据您的业务场景指定合适的内存使用限制。

用于限制内存使用的参数因加载方法而异。有关更多信息，请参见[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)和[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。请注意，加载作业通常在多个BE上运行。因此，这些参数限制了每个涉及的BE上的每个加载作业的内存使用，而不是涉及的所有BE上的加载作业的总内存使用。

StarRocks还提供了参数，供您限制在每个单独的BE上运行的所有加载作业的总内存使用。有关详细信息，请参见本主题的"[系统配置](../loading/Loading_intro.md#system-configurations)"部分。

## 使用注意事项

### 在加载数据时自动填充目标列

在加载数据时，您可以选择不从数据文件的特定字段加载数据：

- 如果在创建StarRocks表时，您为目标StarRocks表列映射的源字段指定了`DEFAULT`关键字，当您加载数据时，StarRocks会自动将指定的默认值填充到目标列中。

  [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)和[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)支持`DEFAULT current_timestamp`、`DEFAULT <default_value>`和`DEFAULT (<expression>)`。[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)仅支持`DEFAULT current_timestamp`和`DEFAULT <default_value>`。

  > **注意**
  >
  > `DEFAULT (<expression>)`仅支持函数`uuid()`和`uuid_numeric()`。

- 如果在创建StarRocks表时，您没有为目标StarRocks表列映射的源字段指定`DEFAULT`关键字，StarRocks会自动在目标列中填充`NULL`。

  > **注意**
  >
  > 如果目标列定义为`NOT NULL`，则加载将失败。

  对于[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)和[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)，您还可以使用指定列映射的参数来指定要填充到目标列中的值。

有关`NOT NULL`和`DEFAULT`的使用，请参见[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

### 为数据加载设置写入仲裁

如果您的StarRocks集群具有多个数据副本，您可以为表设置不同的写入仲裁，即在StarRocks确定加载任务成功之前需要多少个副本。您可以通过在[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)中添加属性`write_quorum`来指定写入仲裁，或者使用[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)为现有表添加此属性。此属性从v2.5开始受支持。

## 系统配置

本节描述了适用于StarRocks提供的所有加载方法的一些参数配置。

### FE配置

您可以在每个FE的配置文件**fe.conf**中配置以下参数：

- `max_load_timeout_second`和`min_load_timeout_second`

  这些参数指定每个加载作业的最大超时期限和最小超时期限。超时期限以秒为单位。默认的最大超时期限为3天，最小超时期限为1秒。您指定的最大超时期限和最小超时期限必须在1秒至3天的范围内。这些参数对同步加载作业和异步加载作业均有效。

- `desired_max_waiting_jobs`

  此参数指定队列中可以等待的最大加载作业数量。默认值为**1024**（v2.4及更早版本为100，v2.5及更高版本为1024）。当FE上处于**PENDING**状态的加载作业数量达到您指定的最大数量时，FE将拒绝新的加载请求。此参数仅对异步加载作业有效。

- `max_running_txn_num_per_db`

  此参数指定您的StarRocks集群中每个数据库允许运行的加载事务的最大数量。一个加载作业可以包含一个或多个事务。默认值为**100**。当数据库中运行的加载事务数量达到您指定的最大数量时，不会调度您提交的后续加载作业。在这种情况下，如果提交同步加载作业，则该作业将被拒绝。如果您提交异步加载作业，则作业将被保持在队列中等待。

  > **注意**
  >
  > StarRocks将所有加载作业一视同仁，并不区分同步加载作业和异步加载作业。

- `label_keep_max_second`

  此参数指定已完成并处于**FINISHED**或**CANCELLED**状态的加载作业的历史记录的保留期限。默认的保留期限为3天。此参数适用于同步加载作业和异步加载作业。

### BE配置

您可以在每个BE的配置文件**be.conf**中配置以下参数：

- `write_buffer_size`
该参数指定最大内存块大小。默认大小为100 MB。加载的数据首先写入BE上的内存块。当加载的数据量达到您指定的最大内存块大小时，数据将被刷新到磁盘。您必须根据业务场景指定适当的最大内存块大小。

- 如果最大内存块大小过小，BE上可能会生成大量小文件。在这种情况下，查询性能会下降。您可以增加最大内存块大小以减少生成的文件数量。
- 如果最大内存块大小过大，远程过程调用（RPC）可能会超时。在这种情况下，您可以根据业务需求调整该参数的值。

- `streaming_load_rpc_max_alive_time_sec`

  每个Writer进程的等待超时周期。默认值为600秒。在数据加载过程中，StarRocks启动一个Writer进程，从每个tablet接收数据并将数据写入。如果一个Writer进程在您指定的等待超时周期内没有收到任何数据，则StarRocks将停止该Writer进程。当StarRocks集群以较低速度处理数据时，Writer进程可能在很长一段时间内没有收到下一批数据，因此会报告“TabletWriter add batch with unknown id”错误。在这种情况下，您可以增加此参数的值。

- `load_process_max_memory_limit_bytes` 和 `load_process_max_memory_limit_percent`

  这些参数指定每个单独BE上所有加载作业可消耗的最大内存量。StarRocks确定两个参数值中的较小内存消耗作为允许的最终内存消耗。

  - `load_process_max_memory_limit_bytes`: 指定最大内存大小。默认最大内存大小为100 GB。
  - `load_process_max_memory_limit_percent`: 指定最大内存使用。默认值为30%。该参数与`mem_limit`参数不同。`mem_limit`参数指定StarRocks集群的总最大内存使用量，默认值为90% x 90%。如果BE所在计算机的内存容量为M，则每个加载作业可消耗的最大内存量计算如下：`M x 90% x 90% x 30%`。

### 系统变量配置

您可以配置以下[系统变量](../reference/System_variable.md):

- `query_timeout`

  查询超时时长。单位: 秒。值范围: `1` 至 `259200`。默认值: `300`。此变量将作用于当前连接中的所有查询语句，以及INSERT语句。

## 故障排除

有关更多信息，请参见[有关数据加载的常见问题解答](../faq/loading/Loading_faq.md)。