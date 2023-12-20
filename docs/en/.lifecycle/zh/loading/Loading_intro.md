---
displayed_sidebar: English
---

# 数据加载概述

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

数据加载是一个过程，它涉及根据您的业务需求清洗和转换来自各种数据源的原始数据，并将处理后的数据加载到 StarRocks 中，以便进行快速的数据分析。

您可以通过运行加载作业来将数据加载到 StarRocks。每个加载作业都有一个唯一的标签，该标签由用户指定或由 StarRocks 自动生成，用于识别作业。每个标签只能用于一次加载作业。加载作业完成后，其标签不能再用于其他加载作业。只有失败的加载作业的标签可以重复使用。这种机制确保了与特定标签关联的数据只能被加载一次，从而实现至多一次（At-Most-Once）的语义。

StarRocks 提供的所有加载方法都能保证原子性。原子性意味着一个加载作业中的合格数据必须全部成功加载，否则没有任何合格数据会被加载。不会出现部分合格数据加载成功而其他数据未加载的情况。请注意，合格数据不包括因数据类型转换错误等质量问题而被过滤掉的数据。

StarRocks 支持两种通信协议用于提交加载作业：MySQL 和 HTTP。有关每种加载方法支持的协议的更多信息，请参见本主题的[加载方法](../loading/Loading_intro.md#loading-methods)部分。

<InsertPrivNote />


## 支持的数据类型

StarRocks 支持加载所有数据类型的数据。您只需注意一些特定数据类型加载时的限制。更多信息，请参见[数据类型](../sql-reference/sql-statements/data-types/BIGINT.md)。

## 加载模式

StarRocks 支持两种加载模式：同步加载模式和异步加载模式。

> **注意**
> 如果您使用外部程序加载数据，您必须根据业务需求选择最适合的加载模式，然后决定您的加载方法。

### 同步加载

在同步加载模式下，提交加载作业后，StarRocks 会同步运行作业来加载数据，并在作业完成后返回作业结果。您可以根据作业结果判断作业是否成功。

StarRocks 提供两种支持同步加载的方法：[Stream Load](../loading/StreamLoad.md) 和 [INSERT](../loading/InsertInto.md)。

同步加载的过程如下：

1. 创建加载作业。

2. 查看 StarRocks 返回的作业结果。

3. 根据作业结果判断作业是否成功。如果作业结果显示加载失败，您可以重试作业。

### 异步加载

在异步加载模式下，提交加载作业后，StarRocks 会立即返回作业创建结果。

- 如果结果显示作业创建成功，StarRocks 将异步运行作业。但这并不意味着数据已经成功加载。您必须使用语句或命令检查作业状态，然后根据作业状态判断数据是否成功加载。

- 如果结果显示作业创建失败，您可以根据失败信息判断是否需要重试作业。

StarRocks 提供三种支持异步加载的方法：[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) 和 [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)。

异步加载的过程如下：

1. 创建加载作业。

2. 查看 StarRocks 返回的作业创建结果，判断作业是否成功创建。
   a. 如果作业创建成功，继续执行步骤 3。
   b. 如果作业创建失败，返回步骤 1 重试。

3. 使用语句或命令检查作业状态，直到作业状态显示为 **FINISHED** 或 **CANCELLED**。

Broker Load 或 Spark Load 作业的工作流程包括五个阶段，如下图所示。

![Broker Load or Spark Load overflow](../assets/4.1-1.png)

工作流程描述如下：

1. **PENDING**

   作业在队列中等待 FE 调度。

2. **ETL**

   FE 对数据进行预处理，包括清洗、分区、排序和聚合。
      > **注意**
      > 如果作业是 Broker Load 作业，这个阶段会直接结束。

3. **LOADING**

   FE 清洗和转换数据，然后将数据发送到 BE。所有数据加载完成后，数据在队列中等待生效。此时，作业状态仍为 **LOADING**。

4. **FINISHED**

   所有数据生效后，作业状态变为 **FINISHED**。此时，数据可以被查询。**FINISHED** 是作业的最终状态。

5. **CANCELLED**

   在作业状态变为 **FINISHED** 之前，您可以随时取消作业。此外，StarRocks 也可能因加载错误自动取消作业。作业被取消后，状态变为 **CANCELLED**。**CANCELLED** 也是作业的最终状态。

Routine 作业的工作流程如下：

1. 作业从 MySQL 客户端提交给 FE。

2. FE 将作业分解为多个任务，每个任务负责从多个分区加载数据。

3. FE 将任务分配给指定的 BE。

4. BE 执行任务，并在完成后向 FE 报告。

5. FE 根据 BE 的报告生成后续任务，重试失败的任务，或根据需要暂停任务调度。

## 加载方法

StarRocks 提供五种加载方法，以帮助您在不同的业务场景中加载数据：[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Routine Load](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) 和 [INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)。

|加载方法|数据来源|业务场景|每个加载作业的数据量|数据文件格式|加载模式|协议|
|---|---|---|---|---|---|---|
|Stream Load|<ul><li>本地文件</li><li>数据流</li></ul>|从本地文件系统加载数据文件或使用程序加载数据流。|10 GB 或更少|<ul><li>CSV</li><li>JSON</li></ul>|同步|HTTP|
|Broker Load|<ul><li>HDFS</li><li>Amazon S3</li><li>Google GCS</li><li>Microsoft Azure 存储</li><li>阿里云 OSS</li><li>腾讯云 COS</li><li>华为云 OBS</li><li>其他 S3 兼容存储系统（如 MinIO）</li></ul>|从 HDFS 或云存储加载数据。|几十 GB 到上百 GB|<ul><li>CSV</li><li>Parquet</li><li>ORC</li></ul>|异步|MySQL|
|Routine Load|Apache Kafka®|从 Kafka 实时加载数据。|MB 到 GB 的小批量数据|<ul><li>CSV</li><li>JSON</li><li>Avro（自 v3.0.1 起支持）</li></ul>|异步|MySQL|
|Spark Load|<ul><li>HDFS</li><li>Hive</li></ul>|<ul><li>使用 Apache Spark™ 集群从 HDFS 或 Hive 迁移大量数据。</li><li>在使用全局数据字典进行数据去重的同时加载数据。</li></ul>|几十 GB 到 TB|<ul><li>CSV</li><li>ORC（自 v2.0 起支持）</li><li>Parquet（自 v2.0 起支持）</li></ul>|异步|MySQL|
|INSERT INTO SELECT|<ul><li>StarRocks 表</li><li>外部表</li><li>AWS S3</li></ul>**注意**<br />从 AWS S3 加载数据时，仅支持 Parquet 格式或 ORC 格式的文件。|<ul><li>从外部表加载数据。</li><li>在 StarRocks 表之间加载数据。</li></ul>|不固定（数据量根据内存大小而异。）|StarRocks 表|同步|MySQL|
|INSERT INTO VALUES|<ul><li>程序</li><li>ETL 工具</li></ul>|<ul><li>以单条记录的形式插入少量数据。</li><li>使用 JDBC 等 API 加载数据。</li></ul>|少量|SQL|同步|MySQL|

您可以根据业务场景、数据量、数据来源、数据文件格式和加载频率等因素来确定您的加载方法。此外，在选择加载方法时，请注意以下几点：

- 当您从 Kafka 加载数据时，我们建议您使用 [Routine Load](../loading/RoutineLoad.md)。但是，如果数据需要进行多表连接和提取、转换、加载（ETL）操作，您可以使用 Apache Flink® 从 Kafka 读取和预处理数据，然后使用 [flink-connector-starrocks](../loading/Flink-connector-starrocks.md) 将数据加载到 StarRocks。

- 当您从 Hive、Iceberg、Hudi 或 Delta Lake 加载数据时，我们建议您创建 [Hive 目录](../data_source/catalog/hive_catalog.md)、[Iceberg 目录](../data_source/catalog/iceberg_catalog.md)、[Hudi 目录](../data_source/catalog/hudi_catalog.md) 或 [Delta Lake 目录](../data_source/catalog/deltalake_catalog.md)，然后使用 [INSERT](../loading/InsertInto.md) 来加载数据。

- 当您从另一个 StarRocks 集群或 Elasticsearch 集群加载数据时，我们建议您创建 [StarRocks 外部表](../data_source/External_table.md#starrocks-external-table) 或 [Elasticsearch 外部表](../data_source/External_table.md#deprecated-elasticsearch-external-table)，然后使用 [INSERT](../loading/InsertInto.md) 来加载数据。

    > **注意**
    > StarRocks 外部表只支持数据写入，不支持数据读取。

```
您可以配置以下[系统变量](../reference/System_variable.md)：

- `query_timeout`

  查询超时时间。单位：秒。值范围：`1`至`259200`。默认值：`300`。此变量将作用于当前连接中的所有查询语句，以及INSERT语句。

## 故障排除

有关更多信息，请参见[数据加载常见问题解答](../faq/loading/Loading_faq.md)。
您可以配置以下[系统变量](../reference/System_variable.md)：

- `query_timeout`

  查询超时时长。单位：秒。值范围：`1` 到 `259200`。默认值：`300`。该变量将作用于当前连接中的所有查询语句，以及 INSERT 语句。

## 故障排除
