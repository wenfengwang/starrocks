---
displayed_sidebar: English
---

# 数据加载概览

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote

数据加载是一个过程，涉及根据业务需求清洗和转换来自多个数据源的原始数据，并将处理后的数据加载到 StarRocks 中，以便进行快速的数据分析。

您可以通过执行加载作业来将数据加载到 StarRocks。每个加载作业都有一个唯一的标签，该标签由用户指定或由 StarRocks 自动生成，用于识别作业。每个标签只能用于一个加载作业。加载作业完成后，其标签不能再用于其他加载作业。只有加载失败的作业的标签可以重新使用。这一机制确保了与特定标签相关联的数据只被加载一次，实现了至多一次的语义。

StarRocks 提供的所有加载方法都保证了原子性。原子性意味着在一个加载作业中，所有符合条件的数据要么全部成功加载，要么一个都不会加载成功。不会出现部分符合条件的数据加载了而其他数据没有加载的情况。请注意，符合条件的数据不包括因数据类型转换错误等质量问题而被筛选掉的数据。

StarRocks支持两种通信协议，可用于提交加载作业：MySQL和HTTP。有关每种加载方式支持的协议的更多信息，请参见本主题的[加载方法](../loading/Loading_intro.md#loading-methods)部分。

<InsertPrivNote />


## 支持的数据类型

StarRocks支持加载所有类型的数据。您只需注意一些特定数据类型的加载限制。更多信息，请参见[数据类型](../sql-reference/sql-statements/data-types/BIGINT.md)。

## 加载模式

StarRocks 支持两种加载模式：同步加载和异步加载。

> **注意**
> 如果您通过外部程序加载数据，请在决定采用的加载方法之前，选择最适合您业务需求的加载模式。

### 同步加载

在同步加载模式下，提交加载作业后，StarRocks 同步执行该作业来加载数据，并在作业完成后返回结果。您可以根据作业结果来判断作业是否成功。

StarRocks提供了两种支持同步加载的方法：[Stream Load](../loading/StreamLoad.md)和[INSERT](../loading/InsertInto.md)。

同步加载的过程如下：

1. 创建加载作业。

2. 查看 StarRocks 返回的作业结果。

3. 根据作业结果判断作业是否成功。如果结果显示加载失败，可以重试作业。

### 异步加载

在异步加载模式下，提交加载作业后，StarRocks 立即返回作业创建结果。

- 如果结果显示作业创建成功，StarRocks 将异步执行该作业。但这并不意味着数据已经成功加载。您必须使用语句或命令来检查作业状态，然后根据作业状态来确定数据是否成功加载。

- 如果结果显示作业创建失败，您可以根据失败信息来决定是否需要重试作业。

StarRocks 提供了三种支持异步加载的方法：[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 和 [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

异步加载的过程如下：

1. 创建加载作业。

2. 查看 StarRocks 返回的作业创建结果，并判断作业是否成功创建。如果作业创建成功，进入步骤 3。如果作业创建失败，返回步骤 1。

3. 使用语句或命令检查作业状态，直到作业状态显示为 **FINISHED** 或 **CANCELLED**。

Broker Load 或 Spark Load 作业的工作流程包括以下五个阶段，如下图所示。

![Broker Load or Spark Load overflow](../assets/4.1-1.png)

工作流程描述如下：

1. **PENDING**（等待中）

   作业在队列中等待 FE 调度。

2. **ETL**（提取、转换、加载）

   FE 预处理数据，包括清洗、分区、排序和聚合。
      > **注意**
      > 如果作业是 **Broker Load**，此阶段会直接完成。

3. **加载中**

   The FE清洗和转换数据，然后将数据发送到BE。所有数据加载完成后，数据在队列中等待生效。此时，作业状态保持为**LOADING**。

4. **FINISHED**（完成）

   当所有数据生效后，作业状态变为 **FINISHED**。此时，数据可以被查询。**FINISHED** 是作业的最终状态。

5. **CANCELLED**（已取消）

   在作业状态变为 **FINISHED** 之前，您可以随时取消作业。此外，StarRocks 可以在加载错误的情况下自动取消作业。作业被取消后，其状态变为 **CANCELLED**。**CANCELLED** 也是作业的最终状态。

Routine 作业的工作流程如下：

1. 作业从 MySQL 客户端提交到 FE。

2. FE 将作业分解为多个任务，每个任务负责从多个分区加载数据。

3. FE 将任务分配给指定的 BE。

4. BE 执行任务，并在任务完成后向 FE 报告。

5. FE 根据 BE 的报告生成后续任务，重试失败的任务或暂停任务调度。

## 加载方法

StarRocks 提供五种加载方法，以适应不同的业务场景：[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 和 [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

|加载方式|数据来源|业务场景|每个加载作业的数据量|数据文件格式|加载方式|协议|
|---|---|---|---|---|---|---|
|流式加载|本地文件数据流|从本地文件系统加载数据文件或使用程序加载数据流。|10 GB 或更少|CSVJSON|同步|HTTP|
|Broker加载|HDFSAmazon S3Google GCSMicrosoft Azure存储阿里云OSST腾讯云COS华威云OBS其他S3兼容存储系统（如MinIO）|从HDFS或云存储加载数据。|几十GB到上百GB|CSVParquetORC|异步|MySQL|
|例行加载|Apache Kafka®|从 Kafka 实时加载数据。|MB 到 GB 的小批量数据|CSVJSONAvro（自 v3.0.1 起支持）|异步|MySQL|
|Spark Load|HDFSHive|使用 Apache Spark™ 集群从 HDFS 或 Hive 迁移大量数据。在使用全局数据字典进行重复数据删除的同时加载数据。|数十 GB 到 TB|CSVORC（自 v2.0 起支持）Parquet (自 v2.0 起支持)|异步|MySQL|
|INSERT INTO SELECT|StarRocks 表外部表AWS S3注意从 AWS S3 加载数据时，仅支持 Parquet 格式或 ORC 格式的文件。|从外部表加载数据。在 StarRocks 表之间加载数据。|不固定（数据量因情况而异）内存大小。）|StarRocks 表|同步|MySQL|
|INSERT INTO VALUES|程序ETL 工具|插入少量数据作为单独的记录。使用 JDBC 等 API 加载数据。|少量|SQL|同步|MySQL|

您可以根据业务场景、数据量、数据来源、数据文件格式和加载频率来确定适合您的加载方法。此外，在选择加载方法时，请注意以下几点：

- 当您从 Kafka 加载数据时，我们推荐您使用 [Routine Load](../loading/RoutineLoad.md)。但如果数据需要进行多表连接和 extract, transform and load (ETL) 操作，您可以使用 Apache Flink® 从 Kafka 读取并预处理数据，然后使用 [flink-connector-starrocks](../loading/Flink-connector-starrocks.md) 将数据加载到 StarRocks。

- 当您从 Hive、Iceberg、Hudi 或 Delta Lake 加载数据时，我们建议您创建相应的目录，然后使用[Hive目录](../data_source/catalog/hive_catalog.md)，[Iceberg目录](../data_source/catalog/iceberg_catalog.md)，[Hudi目录](../data_source/catalog/hudi_catalog.md)，或[Delta Lake目录](../data_source/catalog/deltalake_catalog.md)和然后使用[INSERT](../loading/InsertInto.md)来加载数据。

- 当您从另一个 **StarRocks** 集群或 **Elasticsearch** 集群加载数据时，我们建议您创建一个 [**StarRocks 外部表**](../data_source/External_table.md#starrocks-external-table) 或一个 [**Elasticsearch 外部表**](../data_source/External_table.md#deprecated-elasticsearch-external-table) ，然后使用 [**INSERT**](../loading/InsertInto.md) 来加载数据。

    > **注意**
    > StarRocks 外部表仅支持数据**写入**，不支持数据**读取**。

- 当您需要从MySQL数据库加载数据时，我们建议您创建[MySQL外部表](../data_source/External_table.md#deprecated-mysql-external-table)，然后使用[INSERT](../loading/InsertInto.md)语句来加载数据。如果您希望实时加载数据，建议您按照[Realtime synchronization from MySQL](../loading/Flink_cdc_load.md)中的指南进行操作。

- When you load data from other data sources such as Oracle, PostgreSQL, and SQL Server, we recommend that you create a [JDBC external table](../data_source/External_table.md#external-table-for-a-jdbc-compatible-database) and then use [INSERT](../loading/InsertInto.md) to load the data.

当您从Oracle、PostgreSQL、SQL Server等其他数据源加载数据时，建议您创建一个JDBC外部表，然后使用INSERT语句来加载数据。

![Data loading sources](../assets/4.1-3.png)

## Memory limits

下图概述了StarRocks支持的各种数据源，以及您可以使用的从这些数据源加载数据的方法。

The parameters that are used to limit memory usage vary for each loading method. For more information, see [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md), [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md), [Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md), [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md), and [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md). Note that a load job usually runs on multiple BEs. Therefore, the parameters limit the memory usage of each load job on each involved BE rather than the total memory usage of the load job on all involved BEs.

StarRocks也提供参数，用于限制在每个单独的BE上运行的所有加载作业的总内存使用情况。有关更多信息，请参阅本主题的"[系统配置](../loading/Loading_intro.md#system-configurations)"部分。

## StarRocks提供了参数，用以限制每个加载作业的内存使用量，以此减少内存消耗，特别是在高并发的场景下。但是，请不要设置过低的内存使用限制。如果内存使用限制设置得过低，可能会导致数据因为达到加载作业的内存限制而频繁地从内存中刷新到磁盘。我们建议您根据您的业务场景设定一个合理的内存使用限制。

### Automatically fill in the destination column while loading

用于限制内存使用的参数会根据不同的加载方法而有所不同。更多详细信息，请参见流加载、代理加载、例程加载、Spark加载和INSERT。请注意，加载作业通常会在多个BE上运行。因此，这些参数限制的是每个涉及BE上每个加载作业的内存使用量，而不是所有涉及BE上加载作业的总内存使用量。

- If you have specified the `DEFAULT` keyword for the destination StarRocks table column mapping the source field when you create the StarRocks table, StarRocks automatically fills the specified default value into the destination column.

  StarRocks还提供了参数，用于限制每个单独BE上运行的所有加载作业的总内存使用量。更多详细信息，请参见本主题的“系统配置”部分。

    > **NOTE**
    > `DEFAULT (<expression>)` 仅支持函数 `uuid()` 和 `uuid_numeric()`。

- 在加载数据时自动填充目标列

    > **当您加载数据时，您可以选择不从数据文件的特定字段中加载数据：**
    > 如果目标列被定义为`NOT NULL`，则加载将失败。

  对于[流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[代理加载](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[例程加载](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)和[Spark加载](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)，您还可以使用用于指定列映射的参数来指定要填充到目标列中的值。

注意：`DEFAULT` (`\<expression\>`)只支持函数`uuid()`和`uuid_numeric()`。

### - 如果您在创建StarRocks表时没有为映射源字段的目标列指定DEFAULT关键字，StarRocks将自动填充NULL到目标列。

如果你的StarRocks集群有多个数据副本，您可以为表设置不同的写仲裁，即在StarRocks确定加载任务成功之前需要多少副本返回加载成功。您可以通过在[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)时添加`write_quorum`属性，或者使用[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)为现有表添加此属性来指定写仲裁。从v2.5开始支持此属性。

## - 对于流加载、代理加载、例程加载和Spark加载，您还可以通过指定列映射的参数来设定填充到目标列中的值。

更多关于NOT NULL和DEFAULT的使用信息，请参阅CREATE TABLE文档。

### FE configurations

在每个FE的配置文件**fe.conf**中，您可以配置以下参数：

- 如果您的StarRocks集群有多个数据副本，您可以为表设置不同的写仲裁，即在StarRocks确定加载任务成功之前需要多少副本返回加载成功。您可以在创建表时通过添加write_quorum属性来指定写仲裁，或者使用ALTER TABLE为现有表添加此属性。该属性从v2.5版本开始支持。

  These parameters specify the maximum timeout period and minimum timeout period of each load job. The timeout periods are measured in seconds. The default maximum timeout period spans 3 days, and the default minimum timeout period spans 1 second. The maximum timeout period and minimum timeout period that you specify must fall within the range of 1 second to 3 days. These parameters are valid for both synchronous load jobs and asynchronous load jobs.

- 系统配置

  本节介绍了一些适用于StarRocks提供的所有加载方法的参数配置。这个参数指定了可以在队列中等待的最大加载作业数。默认值为**1024**（在v2.4及更早版本为100，在v2.5及更高版本为1024）。当在FE上处于**PENDING**状态的加载作业数达到您指定的最大数时，FE将拒绝新的加载请求。此参数仅对异步加载作业有效。

- `max_running_txn_num_per_db`

  This parameter specifies the maximum number of ongoing load transactions that are allowed in each database of your StarRocks cluster. A load job can contain one or more transactions. The default value is **100**. When the number of load transactions running in a database reaches the maximum number that you specify, the subsequent load jobs that you submit are not scheduled. In this situation, if you submit a synchronous load job, the job is rejected. If you submit an asynchronous load job, the job is held waiting in queue.

    > **注意**，您可以在每个FE的配置文件`fe.conf`中配置以下参数：
    > StarRocks将所有加载作业一视同仁，并不区分同步加载作业和异步加载作业。

- - desired_max_waiting_jobs：此参数指定可以在队列中等待的最大加载作业数量。默认值为1024（在v2.4及更早版本中为100，v2.5及以后版本为1024）。当FE上处于PENDING状态的加载作业数量达到您指定的最大数量时，FE会拒绝新的加载请求。此参数仅适用于异步加载作业。

  此参数指定了已经完成并处于**FINISHED**或**CANCELLED**状态的加载作业的历史记录保留期。默认的保留期为3天。此参数适用于同步加载作业和异步加载作业。

###   注意：StarRocks统计所有加载作业，不区分同步和异步加载作业。

您可以在每个BE的配置文件**be.conf**中配置以下参数：

- `write_buffer_size`

  BE配置

  - 您可以在每个BE的配置文件be.conf中配置以下参数：
  - - write_buffer_size：此参数指定了最大内存块大小，默认为100MB。加载的数据首先写入BE上的内存块。当加载的数据量达到您指定的最大内存块大小时，数据会被刷新到磁盘。您需要根据业务场景设定合适的最大内存块大小。

-   如果最大内存块大小设置过小，可能会在BE上生成大量小文件，影响查询性能。此时，您可以增加最大内存块大小以减少文件数量。

    如果最大内存块大小设置过大，可能会导致远程过程调用（RPC）超时。这种情况下，您可以根据业务需求调整此参数的值。

- - streaming_load_rpc_max_alive_time_sec：指定了每个Writer进程的等待超时时间，默认为600秒。在数据加载过程中，StarRocks启动Writer进程来从每个Tablet接收和写入数据。如果Writer进程在指定的等待超时时间内没有收到任何数据，StarRocks会停止该进程。当StarRocks集群处理数据速度较慢时，Writer进程可能在较长时间内未收到下一批数据，从而报告“TabletWriter add batch with unknown id”错误。这种情况下，您可以增加此参数的值。

  - load_process_max_memory_limit_bytes和load_process_max_memory_limit_percent：这些参数指定了每个单独BE上所有加载作业可消耗的最大内存量。StarRocks会将这两个参数值中较小的一个作为允许的最终内存消耗量。

-     load_process_max_memory_limit_bytes：指定了最大内存大小，默认为100GB。
-     load_process_max_memory_limit_percent：指定了最大内存使用率，默认为30%。该参数与mem_limit参数不同。mem_limit参数指定了StarRocks集群的总最大内存使用率，默认为90% × 90%。
      如果BE所在机器的内存容量为M，则加载作业的最大可消耗内存量计算公式为：M × 90% × 90% × 30%。

### System variable configurations

你可以配置以下[系统变量](../reference/System_variable.md)：

- 您可以配置以下系统变量：

  - query_timeout：查询超时时长，单位为秒，值范围为1至259200，默认值为300。该变量适用于当前连接中的所有查询语句，包括INSERT语句。

## 问题排查

如需了解更多信息，请参考[数据加载相关的常见问题解答](../faq/loading/Loading_faq.md)。
