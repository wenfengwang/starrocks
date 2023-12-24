---
displayed_sidebar: English
---

# 数据加载概览

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

数据加载是根据业务需求对各种数据源的原始数据进行清洗和转换，然后将结果加载到StarRocks中，以实现快速数据分析。

您可以通过运行加载作业将数据加载到StarRocks中。每个加载作业都有一个唯一的标签，该标签由用户指定或由StarRocks自动生成，用于标识该作业。每个标签只能用于一个加载作业。加载作业完成后，其标签不能重复用于任何其他加载作业。只有失败的加载作业的标签可以被重用。这种机制有助于确保与特定标签关联的数据只能加载一次，从而实现至多一次语义。

StarRocks提供的所有加载方法都可以保证原子性。原子性意味着加载作业中的合格数据必须全部成功加载，或者没有成功加载任何合格数据。绝不会出现部分合格数据被加载而其他数据未加载的情况。需要注意的是，合格数据不包括因数据类型转换错误等质量问题而被筛除的数据。

StarRocks支持两种通信协议：MySQL和HTTP。有关每种加载方法支持的协议的详细信息，请参阅本主题的[加载方法](../loading/Loading_intro.md#loading-methods)部分。

<InsertPrivNote />

## 支持的数据类型

StarRocks支持加载所有数据类型的数据。您只需要注意加载少数特定数据类型时的限制。有关详细信息，请参阅[数据类型](../sql-reference/sql-statements/data-types/BIGINT.md)。

## 加载模式

StarRocks支持两种加载模式：同步加载模式和异步加载模式。

> **注意**
>
> 如果使用外部程序加载数据，则必须在确定加载方式之前选择最适合业务需求的加载模式。

### 同步加载

在同步加载模式下，您提交加载作业后，StarRocks会同步运行该作业以加载数据，并在作业完成后返回作业的结果。您可以根据作业结果检查作业是否成功。

StarRocks提供了两种支持同步加载的加载方法：[Stream Load](../loading/StreamLoad.md)和[INSERT](../loading/InsertInto.md)。

同步加载的流程如下：

1. 创建加载作业。

2. 查看StarRocks返回的作业结果。

3. 根据作业结果检查作业是否成功。如果作业结果指示加载失败，您可以重试该作业。

### 异步加载

在异步加载模式下，您提交加载作业后，StarRocks会立即返回作业创建结果。

- 如果结果显示作业创建成功，StarRocks会异步运行作业。但是，这并不意味着数据已成功加载。您必须使用语句或命令来检查作业的状态。然后，您可以根据作业状态判断数据是否加载成功。

- 如果结果显示作业创建失败，您可以根据失败信息判断是否需要重试作业。

StarRocks提供了三种支持异步加载的加载方法：[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)和[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

异步加载的过程如下：

1. 创建加载作业。

2. 查看StarRocks返回的作业创建结果，判断作业是否创建成功。
   a. 如果作业创建成功，请转到步骤3。
   b. 如果作业创建失败，请返回步骤1。

3. 使用语句或命令检查作业的状态，直到作业状态显示**FINISHED**或**CANCELLED**。

Broker Load或Spark Load作业的工作流程包括五个阶段，如下图所示。

![Broker Load或Spark Load流程图](../assets/4.1-1.png)

工作流程说明如下：

1. **PENDING**

   作业在队列中等待FE调度。

2. **ETL**

   FE对数据进行预处理，包括清洗、分区、排序和聚合。
   > **注意**
   >
   > 如果作业是Broker Load作业，则此阶段直接完成。

3. **LOADING**

   FE对数据进行清洗和转换，然后将数据发送到BE。加载完所有数据后，数据在队列中等待生效。此时，作业的状态仍为**LOADING**。

4. **FINISHED**

   当所有数据生效时，作业的状态变为**FINISHED**。此时，可以查询数据。**FINISHED**是最终作业状态。

5. **CANCELLED**

   在作业状态变为**FINISHED**之前，您可以随时取消作业。此外，StarRocks还可以在出现加载错误时自动取消作业。取消作业后，作业的状态变为**CANCELLED**。**CANCELLED**也是最终作业状态。

例程作业的工作流程说明如下：

1. 作业从MySQL客户端提交到FE。

2. FE将作业拆分为多个任务。每个任务都设计为从多个分区加载数据。

3. FE将任务分发给指定的BE。

4. BE执行任务，并在完成任务后向FE报告。

5. FE会生成后续任务，重试失败的任务，或根据BE的报告暂停任务调度。

## 加载方法

StarRocks提供五种加载方法，帮助您在各种业务场景下加载数据：[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)和[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

| 加载方法     | 数据源                                        | 业务场景                                            | 每个加载作业的数据量                                     | 数据文件格式                                | 加载模式 | 协议 |
| ------------------ | -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------- | ------------ | -------- |
| 流加载        |  <ul><li>本地文件</li><li>数据流</li></ul>| 从本地文件系统加载数据文件或使用程序加载数据流。 | 10 GB或更少                             |<ul><li>CSV</li><li>JSON</li></ul>               | 同步  | HTTP     |
| 代理负载        | <ul><li>HDFS</li><li>Amazon S3</li><li>Google GCS</li><li>Microsoft Azure Storage</li><li>Alibaba Cloud OSS</li><li>Tencent Cloud COS</li><li>Huawei Cloud OBS</li><li>其他兼容S3的存储系统（如MinIO）</li></ul>| 从HDFS或云存储加载数据。                        | 几十GB到几百GB                               | <ul><li>CSV</li><li>Parquet</li><li>ORC</li></ul>| 异步 | MySQL    |
| 例行负载       | Apache Kafka®                                       | 从Kafka实时加载数据。                   | MB到GB的数据量（以小批量形式）                           |<ul><li>CSV</li><li>JSON</li><li>Avro（自v3.0.1起支持）</li></ul>          | 异步 | MySQL    |
| 火花负载         | <ul><li>HDFS</li><li>Hive</li></ul>     |<ul><li>使用Apache Spark™集群从HDFS或Hive迁移大量数据。</li><li>在使用全局数据字典进行去重时加载数据。</li></ul>| 几十GB到TB                                         |<ul><li>CSV</li><li>ORC（自v2.0起支持）<li>Parquet（自v2.0起支持）</li></ul>       | 异步 | MySQL    |
| 插入到选择中 | <ul><li>StarRocks表</li><li>外部表</li><li>AWS S3</li></ul>**注意**<br />当您从AWS S3加载数据时，仅支持Parquet格式或ORC格式的文件。     |<ul><li>从外部表加载数据。</li><li>在StarRocks表之间加载数据。</li></ul>| 不固定（数据量根据内存大小变化） | StarRocks表      | 同步  | MySQL    |
| 插入值 | <ul><li>程序</li><li>ETL工具</li></ul>    |<ul><li>将少量数据作为单个记录插入。</li><li>使用JDBC等接口加载数据。</li></ul>| 少量                                          | SQL                   | 同步  | MySQL    |

您可以根据业务场景、数据量、数据源、数据文件格式和加载频率等因素选择适合自己的加载方法。此外，在选择加载方法时，请注意以下几点：

- 从Kafka加载数据时，建议使用[Routine Load](../loading/RoutineLoad.md)。但是，如果数据需要进行多表连接和ETL操作，您可以使用Apache Flink从Kafka读取和预处理数据，然后使用[flink-connector-starrocks](../loading/Flink-connector-starrocks.md)将数据加载到StarRocks中。

- 从Hive、Iceberg、Hudi或Delta Lake加载数据时，建议创建[Hive目录](../data_source/catalog/hive_catalog.md)、[Iceberg目录](../data_source/catalog/iceberg_catalog.md)、[Hudi目录](../data_source/catalog/hudi_catalog.md)或[Delta Lake目录](../data_source/catalog/deltalake_catalog.md)，然后使用[INSERT](../loading/InsertInto.md)加载数据。

- 当您从另一个 StarRocks 集群或 Elasticsearch 集群加载数据时，我们建议您创建一个[StarRocks 外部表](../data_source/External_table.md#starrocks-external-table)或一个[Elasticsearch 外部表](../data_source/External_table.md#deprecated-elasticsearch-external-table)，然后使用[INSERT](../loading/InsertInto.md)来加载数据。

  > **注意**
  >
  > StarRocks 外部表仅支持数据写入。它们不支持数据读取。

- 当您从 MySQL 数据库加载数据时，我们建议您创建一个[MySQL 外部表](../data_source/External_table.md#deprecated-mysql-external-table)，然后使用[INSERT](../loading/InsertInto.md)来加载数据。如果您想实时加载数据，我们建议您按照[从 MySQL 进行实时同步](../loading/Flink_cdc_load.md)中提供的说明来加载数据。

- 当您从其他数据源（如 Oracle、PostgreSQL 和 SQL Server）加载数据时，我们建议您创建一个[JDBC 外部表](../data_source/External_table.md#external-table-for-a-jdbc-compatible-database)，然后使用[INSERT](../loading/InsertInto.md)来加载数据。

下图概述了StarRocks支持的各种数据源以及您可以使用的加载方法从这些数据源加载数据。

![数据加载源](../assets/4.1-3.png)

## 内存限制

StarRocks提供了一些参数，用于限制每个加载作业的内存使用，从而减少内存消耗，特别是在高并发场景下。但是，请不要指定过低的内存使用限制。如果内存使用限制过低，加载作业的内存使用达到指定限制时，数据可能会频繁地从内存刷新到磁盘。我们建议您根据业务场景设置适当的内存使用限制。

用于限制内存使用的参数因加载方法而异。有关更多信息，请参见[流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker 加载](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[例行加载](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark 加载](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)和[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。请注意，加载作业通常在多个BE上运行。因此，这些参数限制了每个涉及的BE上每个加载作业的内存使用，而不是所有涉及的BE上加载作业的总内存使用。

StarRocks还提供了一些参数，用于限制在每个单独的BE上运行的所有加载作业的总内存使用。有关详细信息，请参见本主题的“[系统配置](../loading/Loading_intro.md#system-configurations)”部分。

## 使用说明

### 在加载时自动填充目标列

在加载数据时，您可以选择不从数据文件的特定字段加载数据：

- 如果您在创建StarRocks表时为目标列映射源字段指定了`DEFAULT`关键字，StarRocks会自动将指定的默认值填充到目标列中。

  [流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker 加载](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[例行加载](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)和[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)支持`DEFAULT current_timestamp`、`DEFAULT <default_value>`和`DEFAULT (<expression>)`。[Spark 加载](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)仅支持`DEFAULT current_timestamp`和`DEFAULT <default_value>`。

  > **注意**
  >
  > `DEFAULT (<expression>)`仅支持函数`uuid()`和`uuid_numeric()`。

- 如果您在创建StarRocks表时没有为目标列映射源字段指定`DEFAULT`关键字，StarRocks会自动将`NULL`填充到目标列中。

  > **注意**
  >
  > 如果目标列定义为`NOT NULL`，加载将失败。

  对于[流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker 加载](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[例行加载](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)和[Spark 加载](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)，您还可以使用用于指定列映射的参数指定要在目标列中填充的值。

有关`NOT NULL`和`DEFAULT`的用法，请参见[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

### 为数据加载设置写入仲裁

如果您的StarRocks集群有多个数据副本，您可以为表设置不同的写入仲裁，即需要多少副本才能返回加载成功，然后StarRocks才能确定加载任务是否成功。您可以通过在[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)时添加`write_quorum`属性来指定写入仲裁，或使用[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)将此属性添加到现有表中。从v2.5开始支持此属性。

## 系统配置

本节描述了适用于StarRocks提供的所有加载方法的一些参数配置。

### FE配置

您可以在每个FE的配置文件**fe.conf**中配置以下参数：

- `max_load_timeout_second`和`min_load_timeout_second`
  
  这些参数指定每个加载作业的最大超时期限和最小超时期限。超时期限以秒为单位。默认最大超时期限为3天，默认最小超时期限为1秒。您指定的最大超时期限和最小超时期限必须在1秒到3天的范围内。这些参数对同步加载作业和异步加载作业都有效。

- `desired_max_waiting_jobs`
  
  此参数指定可以在队列中等待的最大加载作业数。默认值为**1024**（v2.4及更早版本为100，在v2.5及以后版本为1024）。当FE上处于**PENDING**状态的加载作业数量达到您指定的最大数量时，FE将拒绝新的加载请求。此参数仅对异步加载作业有效。

- `max_running_txn_num_per_db`
  
  此参数指定StarRocks集群中每个数据库允许的正在进行的加载事务的最大数量。加载作业可以包含一个或多个事务。默认值为**100**。当数据库中运行的加载事务数达到指定的最大数目时，将不计划您提交的后续加载作业。在这种情况下，如果提交同步加载作业，则该作业将被拒绝。如果提交异步加载作业，则该作业将在队列中等待。

  > **注意**
  >
  > StarRocks将所有加载作业一起统计，并不区分同步加载作业和异步加载作业。

- `label_keep_max_second`
  
  此参数指定已完成且处于**FINISHED**或**CANCELLED**状态的加载作业的历史记录的保留期。默认保留期为3天。此参数对同步加载作业和异步加载作业都有效。

### BE配置

您可以在每个BE的配置文件**be.conf**中配置以下参数：

- `write_buffer_size`
  
  此参数指定最大内存块大小。默认大小为100 MB。加载的数据首先写入BE上的内存块。当加载的数据量达到指定的最大内存块大小时，数据将刷新到磁盘。您必须根据业务场景指定适当的最大内存块大小。

  - 如果最大内存块大小过小，则可能会在BE上生成大量小文件。在这种情况下，查询性能会下降。您可以增加最大内存块大小以减少生成的文件数。
  - 如果最大内存块大小过大，则远程过程调用（RPCs）可能会超时。在这种情况下，您可以根据业务需要调整该参数的值。

- `streaming_load_rpc_max_alive_time_sec`
  
  每个Writer进程的等待超时期限。默认值为600秒。在数据加载过程中，StarRocks会启动一个Writer进程，从每个tablet接收数据并向每个tablet写入数据。如果Writer进程在您指定的等待超时时间内没有收到任何数据，StarRocks会停止该Writer进程。当您的StarRocks集群处理数据速度较慢时，Writer进程可能在很长一段时间内无法接收到下一批数据，因此会报“TabletWriter add batch with unknown id”错误。在这种情况下，您可以增加此参数的值。

- `load_process_max_memory_limit_bytes`和`load_process_max_memory_limit_percent`
  
  这些参数指定每个BE上所有加载作业可消耗的最大内存量。StarRocks确定两个参数值中较小的内存消耗作为允许的最终内存消耗。

  - `load_process_max_memory_limit_bytes`：指定最大内存大小。默认的最大内存大小为100 GB。
  - `load_process_max_memory_limit_percent`：指定最大内存使用量。默认值为30%。此参数与`mem_limit`参数不同。`mem_limit`参数指定StarRocks集群的最大内存总使用量，默认值为90% x 90%。

    如果BE所在的机器的内存容量为M，则加载作业可消耗的最大内存量计算如下：`M x 90% x 90% x 30%`。

### 系统变量配置

您可以配置以下[系统变量](../reference/System_variable.md)：
- `query_timeout`

  查询超时持续时间。单位：秒。取值范围： `1` 到 `259200`。默认值： `300`。此变量将影响当前连接中的所有查询语句，以及 INSERT 语句。

## 故障排除

有关更多信息，请参阅 [有关数据加载的常见问题](../faq/loading/Loading_faq.md)。