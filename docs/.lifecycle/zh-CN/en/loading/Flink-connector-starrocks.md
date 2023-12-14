---
displayed_sidebar: "Chinese"
---

# 从 Apache Flink® 连续加载数据

StarRocks 提供了一个名为 Flink connector 的自研连接器，用于帮助您通过 Flink 将数据加载到 StarRocks 表中。其基本原理是积累数据，然后通过 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 一次性全部加载到 StarRocks 中。

Flink connector 支持 DataStream API、Table API 和 SQL，以及 Python API。它的性能比 Apache Flink® 提供的 [flink-connector-jdbc](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/jdbc/) 更高更稳定。

> **注意**
>
> 使用 Flink connector 将数据加载到 StarRocks 表中需要具有 SELECT 和 INSERT 权限。如果没有这些权限，请按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中提供的说明向您用于连接到 StarRocks 集群的用户授予这些权限。

## 版本要求

| 连接器 | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1 及更高版本| 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1 及更高版本| 8    | 2.11,2.12 |

## 获取 Flink connector

您可以通过以下方式获取 Flink connector JAR 文件：

- 直接下载已编译的 Flink connector JAR 文件。
- 将 Flink connector 作为 Maven 项目的依赖项，然后下载 JAR 文件。
- 自行编译 Flink connector 的源代码为 JAR 文件。

Flink connector JAR 文件的命名格式如下：

- 从 Flink 1.15 版本开始，命名格式为 `flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`。例如，如果安装了 Flink 1.15，并且想要使用 Flink connector 1.2.7，则可以使用 `flink-connector-starrocks-1.2.7_flink-1.15.jar`。

- 在 Flink 1.15 之前，命名格式为 `flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`。例如，如果在您的环境中安装了 Flink 1.14 和 Scala 2.12，并且想要使用 Flink connector 1.2.7，则可以使用 `flink-connector-starrocks-1.2.7_flink-1.14_2.12.jar`。

> **注意**
>
> 通常情况下，Flink connector 的最新版本只与 Flink 的最新三个版本保持兼容。

### 下载已编译的 Jar 文件

直接从 [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) 下载相应版本的 Flink connector Jar 文件。

### Maven 依赖项

在 Maven 项目的 `pom.xml` 文件中，根据以下格式将 Flink connector 作为依赖项添加到项目中。将 `flink_version`、`scala_version` 和 `connector_version` 替换为相应的版本。

- Flink 1.15 和更高版本

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}</version>
    </dependency>
    ```

- Flink 1.15 之前的版本

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}_${scala_version}</version>
    </dependency>
    ```

### 自行编译

1. 下载 [Flink connector package](https://github.com/StarRocks/starrocks-connector-for-apache-flink)。
2. 执行以下命令，将 Flink connector 的源代码编译为 JAR 文件。注意，将 `flink_version` 替换为相应的 Flink 版本。

      ```bash
      sh build.sh <flink_version>
      ```

   例如，如果您的环境中 Flink 的版本是 1.15，则需要执行以下命令：

      ```bash
      sh build.sh 1.15
      ```

3. 进入 `target/` 目录，查找编译后生成的 Flink connector JAR 文件，如 `flink-connector-starrocks-1.2.7_flink-1.15-SNAPSHOT.jar`。

> **注意**
>
> 未正式发布的 Flink connector 名称含有 `SNAPSHOT` 后缀。

## 选项

| **选项**                            | **必需** | **默认值** | **描述**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|--------------------------------------|----------|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| connector                           | 是       | NONE            | 您要使用的连接器。该值必须为 "starrocks"。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| jdbc-url                            | 是       | NONE            | 用于连接到 FE 的 MySQL 服务器的地址。您可以指定多个地址，地址之间必须用逗号 (,) 分隔。格式：`jdbc:mysql://<fe_host1>:<fe_query_port1>,<fe_host2>:<fe_query_port2>,<fe_host3>:<fe_query_port3>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| load-url                            | 是       | NONE            | StarRocks 集群中 FE 的 HTTP URL。您可以指定多个 URL，URL 之间必须用分号 (;) 分隔。格式：`<fe_host1>:<fe_http_port1>;<fe_host2>:<fe_http_port2>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| database-name                       | 是       | NONE            | 要将数据加载到的 StarRocks 数据库的名称。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| table-name                          | 是       | NONE            | 要用来将数据加载到 StarRocks 中的表的名称。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| username                            | 是       | NONE            | 要用来将数据加载到 StarRocks 中的帐户的用户名。 该帐户需要具有 [SELECT 和 INSERT 权限](../sql-reference/sql-statements/account-management/GRANT.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| password                            | 是       | NONE            | 上述帐户的密码。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| sink.semantic                       | 否       | at-least-once   | Sink 保证的语义。有效值：**at-least-once** 和 **exactly-once**。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| sink.version                        | 否       | AUTO           | 用于加载数据的接口。此参数从 Flink connector 版本 1.2.4 起受支持。<ul><li>`V1`：使用 [Stream Load](../loading/StreamLoad.md) 接口加载数据。版本 1.2.4 之前的连接器仅支持此模式。 </li> <li>`V2`：使用 [Stream Load transaction](../loading/Stream_Load_transaction_interface.md) 接口加载数据。需要 StarRocks 至少为版本 2.4。建议使用 `V2`，因为它优化了内存使用，并提供了更稳定的 exactly-once 实现。 </li> <li>`AUTO`：如果 StarRocks 的版本支持事务 Stream Load，将自动选择 `V2`，否则选择 `V1`。</li></ul> |
| sink.label-prefix                   | 否       | NONE           | Stream Load 使用的标签前缀。如果您使用具有连接器 1.2.8 及更高版本的 exactly-once，请建议配置它。参见 [exactly-once 使用注意事项](#exactly-once)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| sink.buffer-flush.max-bytes         | 否       | 94371840(90M)  | 在发送到 StarRocks 之前可以在内存中积累的最大数据大小。最大值范围从 64 MB 到 10 GB。将此参数设置为较大的值可以提高加载性能，但可能会增加加载延迟。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| sink.buffer-flush.max-rows          | 否       | 500000         | 在发送到 StarRocks 之前可以在内存中积累的最大行数。此参数仅在您将 `sink.version` 设置为 `V1` 并将 `sink.semantic` 设置为 `at-least-once` 时可用。有效值：64000 到 5000000。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| 参数                               | 是否必填     | 默认值            | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|------------------------------------|------------|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| sink.buffer-flush.interval-ms      | 否         | 300000            | 刷新数据的间隔。只有当将 `sink.semantic` 设置为 `at-least-once` 时才可用。有效值：1000 到 3600000。单位：毫秒。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.max-retries                   | 否         | 3                 | 系统重试执行流式加载作业的次数。只有当将 `sink.version` 设置为 `V1` 时才可用。有效值：0 到 10。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| sink.connect.timeout-ms            | 否         | 1000              | 建立 HTTP 连接的超时时间。有效值：100 到 60000。单位：毫秒。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.wait-for-continue.timeout-ms  | 否         | 10000             | 1.2.7 版本开始支持。等待 FE 返回 HTTP 100-continue 的超时时间。有效值：`3000` 到 `60000`。单位：毫秒。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.ignore.update-before          | 否         | true              | 1.2.8 版本开始支持。是否忽略从 Flink 加载数据到主键表时来自 StarRocks 的 `UPDATE_BEFORE` 记录。如果将此参数设置为 false，则将该记录视为从 StarRocks 表中的删除操作。                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| sink.properties.*                  | 否         | NONE              | 用于控制流式加载行为的参数。例如，参数 `sink.properties.format` 指定用于流式加载的格式，例如 CSV 或 JSON。有关受支持的参数及其描述的列表，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| sink.properties.format             | 否         | csv               | 用于流式加载的格式。Flink 连接器将在将批数据发送到 StarRocks 之前，将每个批数据转换为指定格式。有效值：`csv` 和 `json`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| sink.properties.row_delimiter      | 否         | \n                | CSV 格式数据的行分隔符。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.properties.column_separator   | 否         | \t                | CSV 格式数据的列分隔符。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.properties.max_filter_ratio   | 否         | 0                 | 流式加载的最大错误容忍度。这是由于数据质量不足而被过滤掉的数据记录的最大百分比。有效值：`0` 到 `1`。默认值：`0`。有关详细信息，请参见 [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.parallelism                   | 否         | NONE              | 连接器的并行度。仅适用于 Flink SQL。如果未设置，则 Flink planner 将决定并行度。在多并行度的场景中，用户需要确保数据按正确顺序编写。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| sink.properties.strict_mode        | 否         | false             | 指定是否启用流式加载的严格模式。影响当存在不合格行（例如不一致的列值）时的加载行为。有效值：`true` 和 `false`。默认值：`false`。有关详细信息，请参见 [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。 |

## Flink 与 StarRocks 之间的数据类型映射

| Flink 数据类型                        | StarRocks 数据类型   |
|-------------------------------------|---------------------|
| BOOLEAN                             | BOOLEAN             |
| TINYINT                             | TINYINT             |
| SMALLINT                            | SMALLINT            |
| INTEGER                             | INTEGER             |
| BIGINT                              | BIGINT              |
| FLOAT                               | FLOAT               |
| DOUBLE                              | DOUBLE              |
| DECIMAL                             | DECIMAL             |
| BINARY                              | INT                 |
| CHAR                                | STRING              |
| VARCHAR                             | STRING              |
| STRING                              | STRING              |
| DATE                                | DATE                |
| TIMESTAMP_WITHOUT_TIME_ZONE(N)      | DATETIME            |
| TIMESTAMP_WITH_LOCAL_TIME_ZONE(N)   | DATETIME            |
| ARRAY&lt;T&gt;                           | ARRAY&lt;T&gt;           |
| MAP&lt;KT,VT&gt;                         | JSON STRING         |
| ROW&lt;arg T...&gt;                      | JSON STRING         |

## 使用注意事项

### 确保一次

- 如果您希望接收端保证一次语义，我们建议您将 StarRocks 升级到 2.5 或更高版本，并将 Flink 连接器升级到 1.2.4 或更高版本。
  - 自 Flink 连接器 1.2.4 版本开始，一次性保证基于 [Stream Load 事务接口](https://docs.starrocks.io/zh-cn/latest/loading/Stream_Load_transaction_interface) 进行了重新设计，该接口从 2.4 版本开始由StarRocks提供。与基于非事务式 Stream Load 非事务接口的以往实现相比，新的实现减少了内存使用和检查点开销，从而增强了实时性能和稳定性。

  - 如果StarRocks版本早于 2.4 或 Flink 连接器版本早于 1.2.4，接收端将自动选择基于非事务式 Stream Load 非事务接口的实现。

- 保证一次的配置

  - `sink.semantic` 的值需要为 `exactly-once`。

  - 如果 Flink 连接器版本为 1.2.8 或更高版本，建议指定 `sink.label-prefix` 的值。请注意，标签前缀必须在 StarRocks 中的所有加载类型（例如 Flink 作业、例行加载和 Broker 加载）中唯一。

    - 如果指定了标签前缀，Flink 连接器将使用标签前缀清理可能在某些 Flink 失败场景中生成的悬挂事务，比如当检查点仍在进行中时 Flink 作业失败。如果使用 `SHOW PROC '/transactions/<db_id>/running';` 命令在 StarRocks 中查看这些事务，一般处于 `PREPARED` 状态。当 Flink 作业从检查点恢复时，Flink 连接器将根据标签前缀和检查点中的某些信息找到这些悬挂事务，并将其中止。当 Flink 作业退出时，由于二阶段提交机制实现了一次性保证，Flink 连接器无法在 Flink 作业退出时中止它们。当 Flink 作业退出时，Flink 连接器尚未收到来自 Flink 检查点协调者的通知，表明这些事务应该包含在成功的检查点中，如果它们无论如何被中止，可能导致数据丢失。您可以在这篇 [blogpost](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/) 中了解有关在 Flink 中实现端到端一次性保证的概览。

    - 如果未指定标签前缀，悬挂事务将仅在超时后由 StarRocks 清理。但是，如果 Flink 作业在事务超时之前经常失败，则运行事务的数量可能达到 StarRocks 的 `max_running_txn_num_per_db` 的限制。超时长度由 StarRocks FE 配置 `prepared_transaction_default_timeout_second` 控制，默认值为 `86400`（1 天）。您可以将该值设置得更小，以便在未指定标签前缀时更快地使事务过期。

- 如果您确定 Flink 作业最终将在长时间停机之后从检查点或保存点中恢复，请相应地调整以下 StarRocks 配置，以避免数据丢失。

  - `prepared_transaction_default_timeout_second`：StarRocks FE 配置，默认值为 `86400`。此配置的值需要大于 Flink 作业的停机时间。否则，在重新启动 Flink 作业之前，包含在成功检查点中的悬挂事务可能会由于超时而中止，这会导致数据丢失。

    请注意，当您为此配置设置较大值时，最好指定 `sink.label-prefix` 的值，以便根据标签前缀和检查点中的某些信息来清理悬挂事务，而不是由于超时而清理（可能导致数据丢失）。

  - `label_keep_max_second` 和 `label_keep_max_num`：StarRocks FE 配置，默认值分别为 `259200` 和 `1000`。有关详细信息，请参见 [FE configurations](../loading/Loading_intro.md#fe-configurations)。`label_keep_max_second` 的值需要大于 Flink 作业的停机时间。否则，Flink 连接器将无法使用保存点或检查点中保存的事务标签来检查 StarRocks 中的事务状态，并找出这些事务是否已提交或尚未提交，这最终可能导致数据丢失。

  这些配置可以通过使用 `ADMIN SET FRONTEND CONFIG` 进行更改：
  
  ```SQL
    ADMIN SET FRONTEND CONFIG ("prepared_transaction_default_timeout_second" = "3600");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "259200");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_num" = "1000");
  ```

### 刷新策略
Flink connector 会在内存中缓冲数据，然后以批处理的方式通过 Stream Load 发送到 StarRocks。触发刷新的方式在至少一次和精确一次之间是不同的。


对于至少一次，刷新将在满足以下任一条件时被触发：

- 缓冲行的字节数达到限制 `sink.buffer-flush.max-bytes`
- 缓冲行的数量达到限制 `sink.buffer-flush.max-rows`。（仅适用于接口版本 V1）
- 自上次刷新以来经过的时间达到限制 `sink.buffer-flush.interval-ms`
- 触发了一个检查点

对于精确一次，刷新只会在触发检查点时发生。

### 监控加载指标

Flink 连接器提供以下指标以监控加载。

| 指标                      | 类型     | 描述                                                             |
|--------------------------|---------|-----------------------------------------------------------------|
| totalFlushBytes          | 计数器  | 成功刷新的字节数。                                              |
| totalFlushRows           | 计数器  | 成功刷新的行数。                                                    |
| totalFlushSucceededTimes | 计数器  | 数据成功刷新的次数。                                      |
| totalFlushFailedTimes    | 计数器  | 数据刷新失败的次数。                                       |
| totalFilteredRows        | 计数器  | 过滤行数，这也包括在 totalFlushRows 中。                      |

## 示例

以下示例展示如何使用 Flink 连接器使用 Flink SQL 或 Flink DataStream 加载数据到 StarRocks 表中。

### 准备工作

#### 创建 StarRocks 表

创建数据库 `test` 并创建一个主键表 `score_board`。

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

- 下载 Flink 二进制文件 [Flink 1.15.2](https://archive.apache.org/dist/flink/flink-1.15.2/flink-1.15.2-bin-scala_2.12.tgz)，并解压到目录 `flink-1.15.2`。
- 下载 [Flink connector 1.2.7](https://repo1.maven.org/maven2/com/starrocks/flink-connector-starrocks/1.2.7_flink-1.15/flink-connector-starrocks-1.2.7_flink-1.15.jar)，并将其放入目录 `flink-1.15.2/lib`。
- 运行以下命令来启动 Flink 集群：

    ```shell
    cd flink-1.15.2
    ./bin/start-cluster.sh
    ```

### 使用 Flink SQL 运行

- 运行以下命令来启动 Flink SQL 客户端。

    ```shell
    ./bin/sql-client.sh
    ```

- 创建一个名为 `score_board` 的 Flink 表，并通过 Flink SQL 客户端将值插入到表中。
请注意，如果要将数据加载到 StarRocks 的主键表中，必须在 Flink DDL 中定义主键。对于其他类型的 StarRocks 表，这是可选的。

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

根据输入记录的类型，例如 CSV Java `String`、JSON Java `String` 或自定义 Java 对象，有几种方法可以实现 Flink DataStream 作业。

- 输入记录为 CSV 格式的 `String`。参见[LoadCsvRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCsvRecords.java)以查看完整示例。

    ```java
    /**
     * 生成 CSV 格式记录。每个记录有三个值，用 "\t" 分隔。
     * 这些值将加载到 StarRocks 表的 `id`、`name` 和 `score` 列中。
     */
    String[] records = new String[]{
            "1\tstarrocks-csv\t100",
            "2\tflink-csv\t100"
    };
    DataStream<String> source = env.fromElements(records);

    /**
     * 使用所需的属性配置连接器。
     * 还需要添加属性 "sink.properties.format" 和 "sink.properties.column_separator"
     * 来告诉连接器输入记录为 CSV 格式，并且列分隔符是 "\t"。
     * 你也可以在 CSV 格式记录中使用其他列分隔符，
     * 但记得相应地修改 "sink.properties.column_separator"。
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
    // 使用这些选项创建一个 sink。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 输入记录为 JSON 格式的 `String`。参见[LoadJsonRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadJsonRecords.java)以查看完整示例。

    ```java
    /**
     * 生成 JSON 格式记录。 
     * 每个记录有三个键值对，对应 StarRocks 表中的 `id`、`name` 和 `score` 列。
     */
    String[] records = new String[]{
            "{\"id\":1, \"name\":\"starrocks-json\", \"score\":100}",
            "{\"id\":2, \"name\":\"flink-json\", \"score\":100}",
    };
    DataStream<String> source = env.fromElements(records);

    /** 
     * 使用所需的属性配置连接器。
     * 还需要添加属性 "sink.properties.format" 和 "sink.properties.strip_outer_array"
     * 来告诉连接器输入记录为 JSON 格式，并且要去除最外层数组结构。 
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
    // 使用这些选项创建一个 sink。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 输入记录为自定义的 Java 对象。参见[LoadCustomJavaRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCustomJavaRecords.java)以查看完整示例。

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
     * Flink连接器将使用Java对象数组（Object[]）来表示要加载到StarRocks表中的行，每个元素是列的值。
     * 您需要定义Object[]的模式，使其与StarRocks表的模式相匹配。
     */
    TableSchema schema = TableSchema.builder()
            .field("id", DataTypes.INT().notNull())
            .field("name", DataTypes.STRING())
            .field("score", DataTypes.INT())
            // 当StarRocks表是主键表时，必须指定notNull()，例如，DataTypes.INT().notNull()，对于主键`id`。
            .primaryKey("id")
            .build();
    // 根据模式将RowData转换为Object[]。
    RowDataTransformer transformer = new RowDataTransformer();
    // 使用模式、选项和转换器创建接收器。
    SinkFunction<RowData> starRockSink = StarRocksSink.sink(schema, options, transformer);
    source.addSink(starRockSink);
    ```

  - 主程序中的`RowDataTransformer`定义如下：


    ```java
    private static class RowDataTransformer implements StarRocksSinkRowBuilder<RowData> {
    
        /**
         * 根据输入的RowData设置对象数组的每个元素。
         * 数组的模式匹配StarRocks表的模式。
         */
        @Override
        public void accept(Object[] internalRow, RowData rowData) {
            internalRow[0] = rowData.id;
            internalRow[1] = rowData.name;
            internalRow[2] = rowData.score;
            // 当StarRocks表是主键表时，需要设置最后一个元素以指示数据加载是更新（UPSERT）还是删除（DELETE）操作。
            internalRow[internalRow.length - 1] = StarRocksSinkOP.UPSERT.ordinal();
        }
    }  

    ```

## 最佳实践

### 向主键表加载数据

本节将展示如何将数据加载到StarRocks主键表中，以实现部分更新和有条件的更新。

您可以查看[通过加载进行数据变更](https://docs.starrocks.io/en-us/latest/loading/Load_to_Primary_Key_tables)了解这些功能的介绍。

这些示例使用Flink SQL。

#### 准备工作


在StarRocks中创建一个名为`test`的数据库，并创建一个名为`score_board`的主键表。

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

此示例将展示如何仅将数据加载到`id`和`name`列。

1. 在MySQL客户端中向StarRocks表`score_board`插入两行数据。

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

2. 在Flink SQL客户端中创建一个名为`score_board`的Flink表。

   - 定义包括列`id`和`name`的DDL。
   - 将选项`sink.properties.partial_update`设置为`true`，指示Flink连接器执行部分更新。
   - 如果Flink连接器版本`<=` 1.2.7，还需要将选项`sink.properties.columns`设置为`id,name,__op`，告诉Flink连接器需要更新哪些列。请注意，您需要在最后追加字段`__op`。字段`__op`表示数据加载是UPSERT还是DELETE操作，并且其值由连接器自动设置。

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
        -- 仅适用于Flink连接器版本 <= 1.2.7
        'sink.properties.columns' = 'id,name,__op'
    ); 
    ```

3. 向Flink表中插入两行数据。数据行的主键与StarRocks表中的行的主键相同，但`name`列中的值已更改。

    ```SQL

    INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'flink-update');

    ```

4. 在MySQL客户端中查询StarRocks表。


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

    您可以看到仅`name`的值发生了变化，而`score`的值没有发生变化。


#### 有条件的更新

此示例将展示如何根据列`score`的值进行有条件的更新。对于`id`的更新仅在`score`的新值大于或等于旧值时才生效。

1. 在MySQL客户端中向StarRocks表中插入两行数据。


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

2. 以以下方式创建一个名为`score_board`的Flink表：

    - 定义包括所有列的DDL。
    - 将选项`sink.properties.merge_condition`设置为`score`，告诉连接器使用列`score`作为条件。
    - 将选项`sink.version`设置为`V1`，告诉连接器使用流式加载。

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
这里我们以UV的计数为例，展示如何将数据加载到`BITMAP`类型的列中。

1. 在MySQL客户端中创建一个StarRocks聚合表。

   在`test`数据库中，创建一个名为`page_uv`的聚合表，其中`visit_users`列被定义为`BITMAP`类型，并配置了聚合函数`BITMAP_UNION`。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT '页面ID',
      `visit_date` datetime NOT NULL COMMENT '访问时间',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT '用户ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. 在Flink SQL客户端中创建一个Flink表。

    Flink表中的`visit_user_id`列属于`BIGINT`类型，我们希望将此列加载到StarRocks表中`BITMAP`类型的`visit_users`列中。因此，在定义Flink表的DDL时，请注意：
    - 因为Flink不支持`BITMAP`，您需要将`visit_user_id`列定义为`BIGINT`类型，以表示StarRocks表中`BITMAP`类型的`visit_users`列。
    - 您需要设置选项`sink.properties.columns`为`page_id,visit_date,user_id,visit_users=to_bitmap(visit_user_id)`，告知连接器Flink表与StarRocks表之间的列映射。并且您需要使用[`to_bitmap`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-functions/bitmap-functions/to_bitmap)函数，告知连接器将`BIGINT`类型的数据转换为`BITMAP`类型。


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

3. 在Flink SQL客户端中将数据加载到Flink表中。

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. 在MySQL客户端中从StarRocks表中计算页面UV。

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

### 将数据加载到HLL类型的列中

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md)可以用于近似计数不同元素，参见[使用HLL进行近似计数](../using_starrocks/Using_HLL.md)。

这里我们以UV的计数为例，展示如何将数据加载到`HLL`类型的列中。

1. 创建StarRocks聚合表。

   在`test`数据库中，创建一个名为`hll_uv`的聚合表，其中`visit_users`列被定义为`HLL`类型，并配置了聚合函数`HLL_UNION`。

    ```SQL
    CREATE TABLE `hll_uv` (
      `page_id` INT NOT NULL COMMENT '页面ID',
      `visit_date` datetime NOT NULL COMMENT '访问时间',
      `visit_users` HLL HLL_UNION NOT NULL COMMENT '用户ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. 在Flink SQL客户端中创建一个Flink表。

   Flink表中的`visit_user_id`列属于`BIGINT`类型，我们希望将此列加载到StarRocks表中`HLL`类型的`visit_users`列中。因此，在定义Flink表的DDL时，请注意：
    - 因为Flink不支持`BITMAP`，您需要将`visit_user_id`列定义为`BIGINT`类型，以表示StarRocks表中`HLL`类型的`visit_users`列。
    - 您需要设置选项`sink.properties.columns`为`page_id,visit_date,user_id,visit_users=hll_hash(visit_user_id)`，告知连接器Flink表与StarRocks表之间的列映射。并且您需要使用[`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md)函数，告知连接器将`BIGINT`类型的数据转换为`HLL`类型。

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

3. 在Flink SQL客户端中将数据加载到Flink表中。

    ```SQL
    INSERT INTO `hll_uv` VALUES
       (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
       (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
       (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. 在MySQL客户端中从StarRocks表中计算页面UV。

    ```SQL
    mysql> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
    **+---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       3 |                           2 |
    |       4 |                           1 |
    +---------+-----------------------------+
    2 rows in set (0.04 sec)
    ```