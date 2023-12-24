---
displayed_sidebar: English
---

# 从 Apache Flink® 持续加载数据

StarRocks 提供了自研的 Flink connector for Apache Flink（简称 Flink connector），帮助您使用 Flink 将数据加载到 StarRocks 表中。基本原理是累积数据，然后通过 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 一次性加载到 StarRocks 中。

Flink connector 支持 DataStream API、Table API & SQL 和 Python API。其性能比 Apache Flink® 提供的 [flink-connector-jdbc](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/jdbc/) 更高、更稳定。

> **注意**
>
> 使用 Flink connector 将数据加载到 StarRocks 表中需要 SELECT 和 INSERT 权限。如果您没有这些权限，请按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中的说明，将这些权限授予您用于连接 StarRocks 集群的用户。

## 版本要求

| 连接器 | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.9 | 1.15,1.16,1.17,1.18 | 2.1 及更高版本| 8 | 2.11,2.12 |
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1 及更高版本| 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1 及更高版本| 8    | 2.11,2.12 |

## 获取 Flink connector

您可以通过以下方式获取 Flink connector 的 JAR 文件：

- 直接下载已编译的 Flink connector JAR 文件。
- 将 Flink connector 作为 Maven 项目的依赖项，然后下载 JAR 文件。
- 自行编译 Flink connector 的源代码成 JAR 文件。

Flink connector JAR 文件的命名格式如下：

- 从 Flink 1.15 开始，命名为 `flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`。例如，如果您安装了 Flink 1.15 并希望使用 Flink connector 1.2.7，则可以使用 `flink-connector-starrocks-1.2.7_flink-1.15.jar`。

- 在 Flink 1.15 之前，命名为 `flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`。例如，如果您在环境中安装了 Flink 1.14 和 Scala 2.12，并且想要使用 Flink connector 1.2.7，则可以使用 `flink-connector-starrocks-1.2.7_flink-1.14_2.12.jar`。

> **注意**
>
> 通常，最新版本的 Flink connector 仅保持与最近三个版本的 Flink 兼容。

### 下载已编译的 Jar 文件

直接从 [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) 下载相应版本的 Flink connector Jar 文件。

### Maven 依赖

在 Maven 项目的 `pom.xml` 文件中，按照以下格式将 Flink connector 添加为依赖项。将 `flink_version`、 `scala_version` 和 `connector_version` 替换为相应的版本。

- 在 Flink 1.15 及更高版本中

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}</version>
    </dependency>
    ```

- 在 Flink 1.15 之前的版本中

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}_${scala_version}</version>
    </dependency>
    ```

### 自行编译

1. 下载 [Flink connector 包](https://github.com/StarRocks/starrocks-connector-for-apache-flink)。
2. 执行以下命令，将 Flink connector 的源代码编译成 JAR 文件。请注意，`flink_version` 需替换为相应的 Flink 版本。

      ```bash
      sh build.sh <flink_version>
      ```

   例如，如果您的环境中的 Flink 版本为 1.15，则需要执行以下命令：

      ```bash
      sh build.sh 1.15
      ```

3. 进入 `target/` 目录，找到编译时生成的 Flink connector JAR 文件，例如 `flink-connector-starrocks-1.2.7_flink-1.15-SNAPSHOT.jar`。

> **注意**
>
> 未正式发布的 Flink connector 名称包含 `SNAPSHOT` 后缀。

## 选项

| **选项**                        | **必填** | **默认值** | **描述**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-----------------------------------|--------------|-------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| connector                         | 是          | 无              | 要使用的连接器。该值必须为 "starrocks"。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| jdbc-url                          | 是          | 无              | 用于连接到 FE 的 MySQL 服务器的地址。 您可以指定多个地址，必须用逗号（,）分隔。格式： `jdbc:mysql://<fe_host1>:<fe_query_port1>,<fe_host2>:<fe_query_port2>,<fe_host3>:<fe_query_port3>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| load-url                          | 是          | 无              | StarRocks 集群中 FE 的 HTTP URL。 您可以指定多个 URL，必须用分号（;）分隔。格式： `<fe_host1>:<fe_http_port1>;<fe_host2>:<fe_http_port2>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| database-name                     | 是          | 无              | 要加载数据的 StarRocks 数据库的名称。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| table-name                        | 是          | 无              | 要将数据加载到 StarRocks 中的表的名称。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| username                          | 是          | 无              | 用于将数据加载到 StarRocks 的帐户的用户名。 该帐户需要 [SELECT 和 INSERT 权限](../sql-reference/sql-statements/account-management/GRANT.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| password                          | 是          | 无              | 上述帐户的密码。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| sink.semantic                     | 否           | at-least-once     | sink 保证的语义。有效值：**at-least-once** 和 **exactly-once**。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| sink.version                      | 否           | AUTO              | 用于加载数据的接口。此参数从 Flink connector 版本 1.2.4 开始支持。<ul><li>`V1`：使用 [Stream Load](../loading/StreamLoad.md) 接口加载数据。1.2.4 之前的连接器仅支持此模式。</li> <li>`V2`：使用 [Stream Load transaction](../loading/Stream_Load_transaction_interface.md) 接口加载数据。要求 StarRocks 至少为 2.4 版本。推荐使用 `V2`，因为它优化了内存使用率，并提供了更稳定的恰好一次实现。 </li> <li>`AUTO`：如果 StarRocks 版本支持事务流加载，则会自动选择 `V2`，否则选择 `V1` </li></ul> |
| sink.label-prefix | 否 | 无 | Stream Load 使用的标签前缀。如果使用连接器 1.2.8 及更高版本的恰好一次，请配置此项。请参阅 [恰好一次使用说明](#exactly-once)。 |
| sink.buffer-flush.max-bytes       | 否           | 94371840（90M）     | 一次发送到 StarRocks 前，内存中可以累积的最大数据量。最大值范围为 64 MB 到 10 GB。将此参数设置为较大的值可以提高加载性能，但可能会增加加载延迟。                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| sink.buffer-flush.max-rows        | 否           | 500000            | 一次发送到 StarRocks 前，内存中可以累积的最大行数。此参数仅在将 `sink.version` 设置为 `V1` 且将 `sink.semantic` 设置为 `at-least-once` 时可用。有效值：64000 到 5000000。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| sink.buffer-flush.interval-ms     | 否           | 300000            | 刷新数据的时间间隔。此参数仅在将 `sink.semantic` 设置为 `at-least-once` 时可用。有效值：1000 到 3600000。单位：毫秒。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| sink.max-retries                  | 否           | 3                 | 系统重试执行 Stream Load 作业的次数。此参数仅在将 `sink.version` 设置为 `V1` 时可用。有效值：0 到 10。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| sink.connect.timeout-ms           | 否           | 30000              | 建立 HTTP 连接的超时时间。有效值：100 到 60000。单位：毫秒。在 Flink connector v1.2.9 之前，默认值为 `1000`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.wait-for-continue.timeout-ms | 否           | 10000             | 从 1.2.7 版本开始支持。等待 FE 的 HTTP 100-continue 响应的超时时间。有效值：3000 到 60000。单位：毫秒。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.ignore.update-before         | 否           | true              | 从 1.2.8 版本开始支持。在将数据加载到主键表时，是否忽略 Flink 的 `UPDATE_BEFORE` 记录。如果将此参数设置为 false，则将该记录视为对 StarRocks 表的删除操作。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.properties.*                 | 否           | 无              | 用于控制 Stream Load 行为的参数。例如，参数 `sink.properties.format` 指定用于 Stream Load 的格式，例如 CSV 或 JSON。有关支持的参数及其说明的列表，请参阅 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。                                                                                                                                                                                                                                                                                                                                 |

| sink.properties.format            | 否           | csv               | 用于流加载的格式。Flink 连接器会将每批数据转换为该格式，然后再发送到 StarRocks。有效值： `csv` 和 `json`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.properties.row_delimiter     | 否           | \n                | CSV 格式数据的行分隔符。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| sink.properties.column_separator  | 否           | \t                | CSV 格式数据的列分隔符。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.properties.max_filter_ratio  | 否           | 0                 | 流加载的最大容错。这是由于数据质量不足而可以过滤掉的数据记录的最大百分比。有效值： `0` 到 `1`。默认值：`0`。有关详细信息，请参阅 [流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。                                                                                                                                                                                                                                                                                                                                                                     |
| 接收器并行性                  | 否           | 无              | 连接器的并行度。仅适用于 Flink SQL。如果未设置，Flink 规划器将决定并行度。在多并行的场景下，用户需要保证数据的写入顺序正确。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| sink.properties.strict_mode | 否 | false | 指定是否启用流加载的严格模式。当存在非限定行（例如不一致的列值）时，它会影响加载行为。有效值： `true` 和 `false`。默认值：`false`。有关详细信息，请参阅 [流加载](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。|

## Flink 和 StarRocks 的数据类型映射关系

| Flink 数据类型                   | StarRocks 数据类型   |
|-----------------------------------|-----------------------|
| BOOLEAN                           | BOOLEAN               |
| TINYINT                           | TINYINT               |
| SMALLINT                          | SMALLINT              |
| INTEGER                           | INTEGER               |
| BIGINT                            | BIGINT                |
| FLOAT                             | FLOAT                 |
| DOUBLE                            | DOUBLE                |
| DECIMAL                           | DECIMAL               |
| BINARY                            | INT                   |
| CHAR                              | STRING                |
| VARCHAR                           | STRING                |
| STRING                            | STRING                |
| DATE                              | DATE                  |
| TIMESTAMP_WITHOUT_TIME_ZONE(N)    | DATETIME              |
| TIMESTAMP_WITH_LOCAL_TIME_ZONE(N) | DATETIME              |
| ARRAY&lt;T&gt;                        | ARRAY&lt;T&gt;              |
| MAP&lt;KT,VT&gt;                        | JSON STRING           |
| ROW&lt;arg T...&gt;                     | JSON STRING           |

## 使用说明

### 正好一次

- 如果希望 sink 保证正好一次的语义，建议将 StarRocks 升级至 2.5 或更高版本，并将 Flink 连接器升级至 1.2.4 或更高版本
  - 从 Flink 连接器 1.2.4 开始，基于 [Stream Load 事务接口](https://docs.starrocks.io/en-us/latest/loading/Stream_Load_transaction_interface) 重新设计了正好一次的保证
    由 StarRocks 自 2.4 版本起提供。相较于之前基于非事务性 Stream Load 的非事务性接口实现，
    新的实现减少了内存使用和检查点开销，从而增强了实时性能和
    负载的稳定性。

  - 如果 StarRocks 版本低于 2.4 或 Flink 连接器版本低于 1.2.4，则 sink
    将根据 Stream Load 非事务性接口自动选择实现。

- 保证正好一次的配置

  - `sink.semantic` 的值需要为 `exactly-once`。

  - 如果 Flink 连接器版本为 1.2.8 及以上，建议指定 `sink.label-prefix` 的值。需要注意的是，标签前缀在 StarRocks 的所有加载类型中必须是唯一的，例如 Flink 作业、Routine Load 和 Broker Load。

    - 如果指定了标签前缀，Flink 连接器将使用标签前缀来清理某些 Flink 中可能生成的延迟事务
      失败场景，例如当检查点仍在进行时，Flink 作业失败。这些挥之不去的交易
      如果您 `PREPARED` 习惯于 `SHOW PROC '/transactions/<db_id>/running';` 在 StarRocks 中查看它们，则通常处于状态。当 Flink 作业从 checkpoint 恢复时，
      Flink 连接器会根据 label 前缀和
      检查点，并中止它们。由于两阶段提交，当 Flink 作业退出时，Flink 连接器无法中止它们
      机制来实现正好一次。当 Flink 作业退出时，Flink 连接器尚未收到来自
      Flink 检查点协调器是否应该将事务包含在一个成功的检查点中，并且它可能
      如果这些事务仍然中止，则会导致数据丢失。您可以大致了解如何实现端到端 - 一次
      在这篇博文的 Flink 中[](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)。

    - 如果未指定标签前缀，则 StarRocks 会在超时后才会清理延迟事务。但是，如果出现以下情况，正在运行的事务数量可以达到 StarRocks 的限制 `max_running_txn_num_per_db` 
      Flink 作业经常在事务超时之前失败。超时长度由 StarRocks FE 配置控制
      `prepared_transaction_default_timeout_second` 默认值为 `86400` （1 天）。您可以为其设置较小的值
      使事务在未指定标签前缀时更快过期。

- 如果您确定 Flink 作业最终会在长时间停机后从检查点或保存点恢复，因为停止或连续故障转移，
  请相应调整以下 StarRocks 配置，以免数据丢失。

  - `prepared_transaction_default_timeout_second`：StarRocks FE 配置，默认值为 `86400`。此配置的值需要大于停机时间
    的 Flink 作业。否则，在重新启动
    Flink 作业，导致数据丢失。

    请注意，当您为此配置设置较大的值时，最好指定 的值`sink.label-prefix`，以便可以根据标签前缀和
      检查点，而不是由于超时（这可能会导致数据丢失）。

  - `label_keep_max_second` 和 `label_keep_max_num`： StarRocks FE 配置，默认值为 `259200` 和 `1000`
    分别。详情请参见 [FE 配置](../loading/Loading_intro.md#fe-configurations)。的值 `label_keep_max_second` 需要大于 Flink 作业的停机时间。否则，Flink 连接器无法通过保存在 Flink 的 savepoint 或 checkpoint 中的事务标签来检查 StarRocks 中的事务状态，也无法判断这些事务是否已提交，最终可能导致数据丢失。

  这些配置是可变的，可以使用以下方法进行修改 `ADMIN SET FRONTEND CONFIG`：

  ```SQL
    ADMIN SET FRONTEND CONFIG ("prepared_transaction_default_timeout_second" = "3600");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "259200");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_num" = "1000");
  ```

### 刷新策略

Flink 连接器会将数据缓冲到内存中，并通过 Stream Load 批量刷新到 StarRocks。如何触发刷新
在至少一次和正好一次之间是不同的。

对于至少一次，当满足以下任一条件时，将触发刷新：

- 缓冲行的字节数达到限制 `sink.buffer-flush.max-bytes`
- 缓冲行数达到限制。 `sink.buffer-flush.max-rows`（仅适用于接收器版本 V1）
- 自上次刷新达到限制以来经过的时间 `sink.buffer-flush.interval-ms`
- 触发检查点

对于正好一次，刷新仅在触发检查点时发生。

### 监控负载指标

Flink 连接器提供以下指标来监控加载情况。

| 度量                     | 类型    | 描述                                                     |
|--------------------------|---------|-----------------------------------------------------------------|
| totalFlushBytes          | 计数器 | 成功刷新的字节数。                                     |
| totalFlushRows           | 计数器 | 成功刷新的行数。                                      |
| totalFlushSucceededTimes | 计数器 | 成功刷新数据的次数。  |
| totalFlushFailedTimes    | 计数器 | 刷新数据失败的次数。                  |
| totalFilteredRows        | 计数器 | 筛选的行数，这也包含在 totalFlushRows 中。    |

### Flink CDC 同步（支持更改 Schema）

[Flink CDC 3.0](https://github.com/ververica/flink-cdc-connectors/releases) 框架可以用来
轻松地 [从 CDC 源构建流式 ELT 流水线](https://ververica.github.io/flink-cdc-connectors/master/content/overview/cdc-pipeline.html)（例如 MySQL 和 Kafka）到 StarRocks。该流水线可以将整个数据库、合并的分片表和 Schema 更改从源同步到 StarRocks。

自 v1.2.9 起，StarRocks 的 Flink 连接器已集成到该框架中，作为 [StarRocks Pipeline Connector](https://ververica.github.io/flink-cdc-connectors/master/content/pipelines/starrocks-pipeline.html)。StarRocks Pipeline Connector 支持：

- 自动创建数据库和表
- 架构更改同步
- 全量和增量数据同步

有关快速入门，请参阅 [使用 Flink CDC 3.0 和 StarRocks Pipeline Connector 从 MySQL 到 StarRocks 进行流式 ELT](https://ververica.github.io/flink-cdc-connectors/master/content/quickstart/mysql-starrocks-pipeline-tutorial.html)。

## 例子

以下示例展示了如何使用 Flink 连接器通过 Flink SQL 或 Flink DataStream 将数据加载到 StarRocks 表中。

### 准备

#### 创建 StarRocks 表

创建数据库 `test` 并创建主键表 `score_board`。

```sql
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
    `id` int(11) NOT NULL COMMENT "",

    `name` varchar(65533) NULL DEFAULT "" COMMENT "",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`);
```

#### 设置 Flink 环境

- 下载 Flink 二进制文件 [Flink 1.15.2](https://archive.apache.org/dist/flink/flink-1.15.2/flink-1.15.2-bin-scala_2.12.tgz)，并将其解压到 `flink-1.15.2` 目录中。
- 下载 [Flink 连接器 1.2.7](https://repo1.maven.org/maven2/com/starrocks/flink-connector-starrocks/1.2.7_flink-1.15/flink-connector-starrocks-1.2.7_flink-1.15.jar)，并将其放入 `flink-1.15.2/lib` 目录。
- 运行以下命令启动 Flink 集群：

    ```shell
    cd flink-1.15.2
    ./bin/start-cluster.sh
    ```

### 使用 Flink SQL 运行

- 运行以下命令启动 Flink SQL 客户端。

    ```shell
    ./bin/sql-client.sh
    ```

- 创建一个 Flink `score_board` 表，并通过 Flink SQL 客户端向表中插入值。
注意，如果要将数据加载到 StarRocks 的主键表中，则必须在 Flink DDL 中定义主键。对于其他类型的 StarRocks 表，这是可选的。

    ```SQL
    CREATE TABLE `score_board` (
        `id` INT,
        `name` STRING,
        `score` INT,
        PRIMARY KEY (id) NOT ENFORCED
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        
        'table-name' = 'score_board',
        'username' = 'root',
        'password' = ''
    );

    INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);
    ```

### 使用 Flink DataStream 运行

根据输入记录的类型，有几种方法可以实现 Flink DataStream 作业，例如 CSV Java `String`、JSON `String` Java 或自定义 Java 对象。

- 输入记录为 CSV 格式`String`。有关 [完整示例，请参阅](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCsvRecords.java) LoadCsvRecords。

    ```java
    /**
     * 生成 CSV 格式的记录。每条记录有三个值，用 "\t" 分隔。
     * 这些值将加载到 StarRocks 表的列 `id`、`name` 和 `score` 中。
     */
    String[] records = new String[]{
            "1\tstarrocks-csv\t100",
            "2\tflink-csv\t100"
    };
    DataStream<String> source = env.fromElements(records);

    /**
     * 使用所需的属性配置连接器。
     * 您还需要添加属性“sink.properties.format”和“sink.properties.column_separator”
     * 告知连接器输入记录为 CSV 格式，列分隔符为 “\t”。
     * 您还可以在 CSV 格式的记录中使用其他列分隔符，
     * 但切记要相应地修改 “sink.properties.column_separator”。
     */
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .withProperty("sink.properties.format", "csv")
            .withProperty("sink.properties.column_separator", "\t")
            .build();
    // 使用选项创建接收器。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 输入记录为 JSON 格式`String`。有关 [完整示例，请参阅](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadJsonRecords.java) LoadJsonRecords。

    ```java
    /**
     * 生成 JSON 格式的记录。
     * 每条记录有三个键值对，对应于 StarRocks 表中的列 `id`、`name` 和 `score`。
     */
    String[] records = new String[]{
            "{\"id\":1, \"name\":\"starrocks-json\", \"score\":100}",
            "{\"id\":2, \"name\":\"flink-json\", \"score\":100}",
    };
    DataStream<String> source = env.fromElements(records);

    /** 
     * 使用所需的属性配置连接器。
     * 您还需要添加属性“sink.properties.format”和“sink.properties.strip_outer_array”
     * 告知连接器输入记录为 JSON 格式，并剥离最外层的数组结构。
     */
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .withProperty("sink.properties.format", "json")
            .withProperty("sink.properties.strip_outer_array", "true")
            .build();
    // 使用选项创建接收器。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 输入记录为自定义 Java 对象。有关 [完整示例，请参阅](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCustomJavaRecords.java) LoadCustomJavaRecords。

  - 在此示例中，输入记录是一个简单的 POJO `RowData`。

      ```java
      public static class RowData {
              public int id;
              public String name;
              public int score;
    
              public RowData() {}
    
              public RowData(int id, String name, int score) {
                  this.id = id;
                  this.name = name;
                  this.score = score;
              }
          }
      ```

  - 主程序如下：

    ```java
    // 生成使用 RowData 作为容器的记录。
    RowData[] records = new RowData[]{
            new RowData(1, "starrocks-rowdata", 100),
            new RowData(2, "flink-rowdata", 100),
        };
    DataStream<RowData> source = env.fromElements(records);

    // 使用所需的属性配置连接器。
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .build();

    /**
     * Flink 连接器将使用 Java 对象数组（Object[]）表示要加载到 StarRocks 表中的行，
     * 每个元素都是一列的值。
     * 您需要定义与 StarRocks 表的 schema 相匹配的 Object[] schema。
     */
    TableSchema schema = TableSchema.builder()
            .field("id", DataTypes.INT().notNull())
            .field("name", DataTypes.STRING())
            .field("score", DataTypes.INT())
            // 当 StarRocks 表为主键表时，您必须指定 notNull()，例如 DataTypes.INT().notNull() 作为主键`id`。
            .primaryKey("id")
            .build();
    // 根据 schema 将 RowData 转换为 Object[]。
    RowDataTransformer transformer = new RowDataTransformer();
    // 使用 schema、options 和 transformer 创建接收器。
    SinkFunction<RowData> starRockSink = StarRocksSink.sink(schema, options, transformer);
    source.addSink(starRockSink);
    ```

  - 主程序中的 `RowDataTransformer` 定义如下：

    ```java
    private static class RowDataTransformer implements StarRocksSinkRowBuilder<RowData> {
    
        /**
         * 根据输入的 RowData 设置对象数组的每个元素。
         * 数组的 schema 与 StarRocks 表的 schema 相匹配。
         */
        @Override
        public void accept(Object[] internalRow, RowData rowData) {
            internalRow[0] = rowData.id;
            internalRow[1] = rowData.name;
            internalRow[2] = rowData.score;
            // 当 StarRocks 表为主键表时，您需要设置最后一个元素，以指示数据加载是 UPSERT 还是 DELETE 操作。
            internalRow[internalRow.length - 1] = StarRocksSinkOP.UPSERT.ordinal();
        }
    }  
    ```

## 最佳实践

### 将数据加载到主键表

本节将展示如何将数据加载到 StarRocks 主键表中，以实现部分更新和条件更新。
您可以查看 [通过加载更改数据](https://docs.starrocks.io/en-us/latest/loading/Load_to_Primary_Key_tables) 了解这些功能的介绍。
这些示例使用 Flink SQL。

#### 准备工作

在 StarRocks 中创建 `test` 数据库，并在其中创建主键表 `score_board`。

```SQL
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
    `id` int(11) NOT NULL COMMENT "",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`);
```

#### 部分更新

此示例将展示如何仅将数据加载到列 `id` 和 `name` 中。

1. 在 MySQL 客户端的 `score_board` StarRocks 表中插入两行数据。

    ```SQL
    mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);

    mysql> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | flink     |   100 |
    +------+-----------+-------+
    2 rows in set (0.02 sec)
    ```

2. 在 Flink SQL 客户端中创建 Flink `score_board` 表。

   - 定义仅包含列 `id` 和 `name` 的 DDL。
   - 设置 `sink.properties.partial_update` 选项为 `true`，告诉 Flink 连接器执行部分更新。

   - 如果 Flink 连接器版本 `<=` 1.2.7，则还需要将选项 `sink.properties.columns` 设置为 `id,name,__op`，以告诉 Flink 连接器需要更新哪些列。请注意，您需要在末尾附加字段 `__op`。字段 `__op` 指示数据加载是 UPSERT 还是 DELETE 操作，其值由连接器自动设置。

    ```SQL
    CREATE TABLE `score_board` (
        `id` INT,
        `name` STRING,
        PRIMARY KEY (id) NOT ENFORCED
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        'table-name' = 'score_board',
        'username' = 'root',
        'password' = '',
        'sink.properties.partial_update' = 'true',
        -- 仅适用于 Flink 连接器版本 <= 1.2.7
        'sink.properties.columns' = 'id,name,__op'
    ); 
    ```

3. 在 Flink 表中插入两行数据。数据行的主键与 StarRocks 表中的行主键相同，但 `name` 列的值已被修改。

    ```SQL
    INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'flink-update');
    ```

4. 在 MySQL 客户端中查询 StarRocks 表。
  
    ```SQL
    mysql> select * from score_board;
    +------+------------------+-------+
    | id   | name             | score |
    +------+------------------+-------+
    |    1 | starrocks-update |   100 |
    |    2 | flink-update     |   100 |
    +------+------------------+-------+
    2 rows in set (0.02 sec)
    ```

    您可以看到，只有 `name` 列的值发生了变化，而 `score` 列的值没有发生变化。

#### 条件更新

此示例将展示如何根据 `score` 列的值进行条件更新。仅当 `id` 的新 `score` 值大于或等于旧值时，更新才会生效。

1. 在 MySQL 客户端的 StarRocks 表中插入两行数据。

    ```SQL
    mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);
    
    mysql> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | flink     |   100 |
    +------+-----------+-------+
    2 rows in set (0.02 sec)
    ```

2. 以以下方式创建 Flink 表 `score_board`：
  
    - 定义包括所有列的 DDL。
    - 将选项 `sink.properties.merge_condition` 设置为 `score`，以告知连接器使用列 `score` 作为条件。
    - 将选项 `sink.version` 设置为 `V1`，指示连接器使用流加载。

    ```SQL
    CREATE TABLE `score_board` (
        `id` INT,
        `name` STRING,
        `score` INT,
        PRIMARY KEY (id) NOT ENFORCED
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        'table-name' = 'score_board',
        'username' = 'root',
        'password' = '',
        'sink.properties.merge_condition' = 'score',
        'sink.version' = 'V1'
        );
    ```

3. 在 Flink 表中插入两行数据。数据行的主键与 StarRocks 表中的行主键相同。第一行数据的 `score` 列具有较小的值，而第二行数据的 `score` 列具有较大的值。

    ```SQL
    INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'flink-update', 101);
    ```

4. 在 MySQL 客户端中查询 StarRocks 表。

    ```SQL
    mysql> select * from score_board;
    +------+--------------+-------+
    | id   | name         | score |
    +------+--------------+-------+
    |    1 | starrocks    |   100 |
    |    2 | flink-update |   101 |
    +------+--------------+-------+
    2 rows in set (0.03 sec)
    ```

   您可以看到，只有第二行数据的值发生了变化，而第一行数据的值没有发生变化。

### 将数据加载到 BITMAP 类型的列中

[`BITMAP`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-statements/data-types/BITMAP) 通常用于加速准确计数非重复值，例如计算 UV，请参阅 [使用位图进行准确计数](https://docs.starrocks.io/en-us/latest/using_starrocks/Using_bitmap)。
这里我们以计算 UV 为例，展示如何将数据加载到 `BITMAP` 类型的列中。

1. 在 MySQL 客户端中创建 StarRocks 聚合表。

   在 `test` 数据库中，创建一个聚合表 `page_uv`，其中列 `visit_users` 被定义为 `BITMAP` 类型，并配置了聚合函数 `BITMAP_UNION`。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT '页面 ID',
      `visit_date` datetime NOT NULL COMMENT '访问时间',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT '用户 ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. 在 Flink SQL 客户端中创建 Flink 表。

    Flink 表中的列 `visit_user_id` 是 `BIGINT` 类型，我们希望将该列加载到 StarRocks 表中 `BITMAP` 类型的列 `visit_users` 中。因此，在定义 Flink 表的 DDL 时，需要注意：
    - 由于 Flink 不支持 `BITMAP`，因此需要将列 `visit_user_id` 定义为 `BIGINT` 类型，以表示 StarRocks 表中 `BITMAP` 类型的列 `visit_users`。
    - 您需要将选项 `sink.properties.columns` 设置为 `page_id,visit_date,user_id,visit_users=to_bitmap(visit_user_id)`，以告知连接器 Flink 表和 StarRocks 表之间的列映射。此外，您需要使用 [`to_bitmap`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-functions/bitmap-functions/to_bitmap)
   函数，以告知连接器将 `BIGINT` 类型的数据转换为 `BITMAP` 类型。

    ```SQL
    CREATE TABLE `page_uv` (
        `page_id` INT,
        `visit_date` TIMESTAMP,
        `visit_user_id` BIGINT
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        'table-name' = 'page_uv',
        'username' = 'root',
        'password' = '',
        'sink.properties.columns' = 'page_id,visit_date,visit_user_id,visit_users=to_bitmap(visit_user_id)'
    );
    ```

3. 在 Flink SQL 客户端中将数据加载到 Flink 表中。

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. 在 MySQL 客户端中从 StarRocks 表中计算页面 UV。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `page_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       2 |                           1 |
    |       1 |                           3 |
    +---------+-----------------------------+
    2 rows in set (0.05 sec)
    ```

### 将数据加载到 HLL 类型的列中

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md) 可用于近似计数非重复值，请参阅 [使用 HLL 进行近似计数非重复值](../using_starrocks/Using_HLL.md)。

这里我们以计算 UV 为例，展示如何将数据加载到 `HLL` 类型的列中。

1. 创建 StarRocks 聚合表

   在 `test` 数据库中，创建一个聚合表 `hll_uv`，其中列 `visit_users` 被定义为 `HLL` 类型，并配置了聚合函数 `HLL_UNION`。

    ```SQL
    CREATE TABLE `hll_uv` (
      `page_id` INT NOT NULL COMMENT '页面 ID',
      `visit_date` datetime NOT NULL COMMENT '访问时间',
      `visit_users` HLL HLL_UNION NOT NULL COMMENT '用户 ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. 在 Flink SQL 客户端中创建 Flink 表。

    Flink 表中的列 `visit_user_id` 是 `BIGINT` 类型，我们希望将该列加载到 StarRocks 表中 `HLL` 类型的列 `visit_users` 中。因此，在定义 Flink 表的 DDL 时，需要注意：
    - 由于 Flink 不支持 `HLL`，因此需要将列 `visit_user_id` 定义为 `BIGINT` 类型，以表示 StarRocks 表中 `HLL` 类型的列 `visit_users`。
    - 您需要设置选项 `sink.properties.columns` 为 `page_id,visit_date,user_id,visit_users=hll_hash(visit_user_id)`，以告知连接器 Flink 表和 StarRocks 表之间的列映射关系。此外，您还需要使用 [`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md) 函数，以告知连接器将 `BIGINT` 类型的数据转换为 `HLL` 类型。

    ```SQL
    CREATE TABLE `hll_uv` (
        `page_id` INT,
        `visit_date` TIMESTAMP,
        `visit_user_id` BIGINT
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        'table-name' = 'hll_uv',
        'username' = 'root',
        'password' = '',
        'sink.properties.columns' = 'page_id,visit_date,visit_user_id,visit_users=hll_hash(visit_user_id)'
    );
    ```


3. 在 Flink SQL 客户端中加载数据到 Flink 表中。

    ```SQL
    INSERT INTO `hll_uv` VALUES
       (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
       (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
       (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. 在 MySQL 客户端中计算页面 UV 从 StarRocks 表中。

    ```SQL
    mysql> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
    **+---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       3 |                           2 |
    |       4 |                           1 |
    +---------+-----------------------------+
    2 行在集合中 (0.04 秒)
    ```
