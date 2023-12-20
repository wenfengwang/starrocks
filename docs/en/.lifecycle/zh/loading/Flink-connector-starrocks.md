---
displayed_sidebar: English
---

# 从 Apache Flink® 持续加载数据

StarRocks 提供了自研的连接器，名为 Flink connector for Apache Flink®（简称 Flink connector），帮助您使用 Flink 将数据加载到 StarRocks 表中。基本原理是累积数据，然后通过 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 将其一次性全部加载到 StarRocks 中。

Flink 连接器支持 DataStream API、Table API & SQL，以及 Python API。它的性能比 Apache Flink® 提供的 [flink-connector-jdbc](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/jdbc/) 更高且更稳定。

> **注意**
> 使用 Flink 连接器将数据加载到 StarRocks 表中需要 SELECT 和 INSERT 权限。如果您没有这些权限，请按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中提供的说明将这些权限授予用于连接到 StarRocks 集群的用户。

## 版本要求

|连接器|Flink|StarRocks|Java|Scala|
|---|---|---|---|---|
|1.2.9|1.15,1.16,1.17,1.18|2.1 及更高版本|8|2.11,2.12|
|1.2.8|1.13,1.14,1.15,1.16,1.17|2.1 及更高版本|8|2.11,2.12|
|1.2.7|1.11,1.12,1.13,1.14,1.15|2.1 及更高版本|8|2.11,2.12|

## 获取 Flink 连接器

您可以通过以下方式获取 Flink Connector JAR 文件：

- 直接下载编译好的 Flink Connector JAR 文件。
- 将 Flink 连接器添加为 Maven 项目中的依赖项，然后下载 JAR 文件。
- 自行将 Flink Connector 的源码编译成 JAR 文件。

Flink Connector JAR 文件的命名格式如下：

- 从 Flink 1.15 开始，它是 `flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`。例如，如果您安装了 Flink 1.15，并且想要使用 Flink Connector 1.2.7，则可以使用 `flink-connector-starrocks-1.2.7_flink-1.15.jar`。

- 在 Flink 1.15 之前，它是 `flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`。例如，如果您的环境中安装了 Flink 1.14 和 Scala 2.12，并且想要使用 Flink Connector 1.2.7，则可以使用 `flink-connector-starrocks-1.2.7_flink-1.14_2.12.jar`。

> **注意**
> 通常，最新版本的 Flink 连接器只维护与 Flink 最新三个版本的兼容性。

### 下载编译好的 Jar 文件

直接从 [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) 下载对应版本的 Flink Connector Jar 文件。

### Maven 依赖

在 Maven 项目的 `pom.xml` 文件中，按照以下格式添加 Flink 连接器作为依赖项。将 `flink_version`、`scala_version` 和 `connector_version` 替换为相应的版本。

- 在 Flink 1.15 及更高版本中

  ```xml
  <dependency>
      <groupId>com.starrocks</groupId>
      <artifactId>flink-connector-starrocks</artifactId>
      <version>${connector_version}_flink-${flink_version}</version>
  </dependency>
  ```

- 在 Flink 1.15 之前的版本

  ```xml
  <dependency>
      <groupId>com.starrocks</groupId>
      <artifactId>flink-connector-starrocks</artifactId>
      <version>${connector_version}_flink-${flink_version}_${scala_version}</version>
  </dependency>
  ```

### 自行编译

1. 下载 [Flink 连接器包](https://github.com/StarRocks/starrocks-connector-for-apache-flink)。
2. 执行以下命令将 Flink Connector 源代码编译成 JAR 文件。请注意，`flink_version` 替换为相应的 Flink 版本。

   ```bash
   sh build.sh <flink_version>
   ```
   例如，如果您环境中的 Flink 版本为 1.15，则需要执行以下命令：

   ```bash
   sh build.sh 1.15
   ```

3. 进入 `target/` 目录，找到编译生成的 Flink Connector JAR 文件，如 `flink-connector-starrocks-1.2.7_flink-1.15-SNAPSHOT.jar`。

> **注意**
> 未正式发布的 Flink 连接器名称中包含 `SNAPSHOT` 后缀。

## 选项

|**选项**|**必需**|**默认值**|**说明**|
|---|---|---|---|
|connector|是|无|您要使用的连接器。该值必须是 "starrocks"。|
|jdbc-url|是|无|用于连接 FE 的 MySQL 服务器的地址。您可以指定多个地址，这些地址必须用逗号 (,) 分隔。格式：`jdbc:mysql://<fe_host1>:<fe_query_port1>,<fe_host2>:<fe_query_port2>,<fe_host3>:<fe_query_port3>`。|
|load-url|是|无|您的 StarRocks 集群中 FE 的 HTTP URL。您可以指定多个 URL，这些 URL 必须用分号 (;) 分隔。格式：`<fe_host1>:<fe_http_port1>;<fe_host2>:<fe_http_port2>`。|
|database-name|是|无|您想要加载数据的 StarRocks 数据库名称。|
|table-name|是|无|您想要用来加载数据到 StarRocks 的表名称。|
|username|是|无|您想要用来加载数据到 StarRocks 的账户用户名。该账户需要 [SELECT 和 INSERT 权限](../sql-reference/sql-statements/account-management/GRANT.md)。|
|password|是|无|上述账户的密码。|
|sink.semantic|否|at-least-once|sink 保证的语义。有效值：**at-least-once** 和 **exactly-once**。|
|sink.version|否|AUTO|用于加载数据的接口。此参数从 Flink Connector 版本 1.2.4 开始支持。<ul><li>`V1`：使用 [Stream Load](../loading/StreamLoad.md) 接口加载数据。1.2.4 之前的连接器只支持此模式。</li> <li>`V2`：使用 [Stream Load 事务](../loading/Stream_Load_transaction_interface.md) 接口加载数据。它要求 StarRocks 至少为版本 2.4。推荐 `V2`，因为它优化了内存使用并提供了更稳定的 exactly-once 实现。</li> <li>`AUTO`：如果 StarRocks 版本支持事务 Stream Load，将自动选择 `V2`，否则选择 `V1`。</li></ul>|
|sink.label-prefix|否|无|Stream Load 使用的标签前缀。如果您在连接器 1.2.8 及更高版本中使用 exactly-once，建议配置此项。参见 [exactly-once 使用说明](#exactly-once)。|
|sink.buffer-flush.max-bytes|否|94371840(90M)|一次性发送到 StarRocks 之前可以在内存中累积的最大数据大小。最大值范围为 64 MB 至 10 GB。将此参数设置为较大值可以提高加载性能，但可能会增加加载延迟。|
|sink.buffer-flush.max-rows|否|500000|一次性发送到 StarRocks 之前可以在内存中累积的最大行数。此参数仅在 `sink.version` 设置为 `V1` 且 `sink.semantic` 设置为 `at-least-once` 时可用。有效值：64000 至 5000000。|
|sink.buffer-flush.interval-ms|否|300000|数据刷新的时间间隔。此参数仅在 `sink.semantic` 设置为 `at-least-once` 时可用。有效值：1000 至 3600000。单位：毫秒。|
|sink.max-retries|否|3|系统尝试执行 Stream Load 作业的次数。此参数仅在 `sink.version` 设置为 `V1` 时可用。有效值：0 至 10。|
|sink.connect.timeout-ms|否|30000|建立 HTTP 连接的超时时间。有效值：100 至 60000。单位：毫秒。在 Flink Connector v1.2.9 之前，默认值为 `1000`。|
|sink.wait-for-continue.timeout-ms|否|10000|自 1.2.7 起支持。等待 FE 的 HTTP 100-Continue 响应的超时时间。有效值：`3000` 至 `60000`。单位：毫秒|
|sink.ignore.update-before|否|true|自版本 1.2.8 起支持。是否忽略 Flink 中的 `UPDATE_BEFORE` 记录，当加载数据到 StarRocks 的主键表时。如果此参数设置为 false，则该记录将被视为对 StarRocks 表的删除操作。|
|sink.properties.*|否|无|用于控制 Stream Load 行为的参数。例如，参数 `sink.properties.format` 指定用于 Stream Load 的格式，如 CSV 或 JSON。有关支持的参数及其描述的列表，请参阅 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。|
|sink.properties.format|否|csv|用于 Stream Load 的格式。Flink 连接器会将每批数据转换为指定格式，然后发送到 StarRocks。有效值：`csv` 和 `json`。|
|sink.properties.row_delimiter|否|\n|CSV 格式数据的行分隔符。|
|sink.properties.column_separator|否|\t|CSV 格式数据的列分隔符。|
|sink.properties.max_filter_ratio|否|0|Stream Load 的最大错误容忍率。它是由于数据质量不足而可以过滤掉的数据记录的最大百分比。有效值：`0` 至 `1`。默认值：`0`。详情请参阅 [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。|
```
|sink.parallelism|否|无|连接器的并行度。仅适用于 Flink SQL。如果没有设置，Flink planner 将决定并行度。在多并行场景下，用户需要保证数据按照正确的顺序写入。|
|sink.properties.strict_mode|否|false|指定是否启用 Stream Load 的严格模式。它会影响加载行为，特别是当存在不合格的行时，例如列值不一致。有效值：`true` 和 `false`。默认值：`false`。有关详细信息，请参阅 [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。|

## Flink 和 StarRocks 之间的数据类型映射

|Flink 数据类型|StarRocks 数据类型|
|---|---|
|BOOLEAN|BOOLEAN|
|TINYINT|TINYINT|
|SMALLINT|SMALLINT|
|INTEGER|INTEGER|
|BIGINT|BIGINT|
|FLOAT|FLOAT|
|DOUBLE|DOUBLE|
|DECIMAL|DECIMAL|
|BINARY|BINARY|
|CHAR|CHAR|
|VARCHAR|VARCHAR|
|STRING|STRING|
|DATE|DATE|
|TIMESTAMP_WITHOUT_TIME_ZONE(N)|DATETIME|
|TIMESTAMP_WITH_LOCAL_TIME_ZONE(N)|DATETIME|
|ARRAY&lt;T&gt;|ARRAY&lt;T&gt;|
|MAP&lt;KT,VT&gt;|JSON STRING|
|ROW&lt;arg T...&gt;|JSON STRING|

## 使用说明

### Exactly Once

- 如果您希望 sink 保证 exactly-once 语义，我们建议您将 StarRocks 升级到 2.5 或更高版本，将 Flink connector 升级到 1.2.4 或更高版本。
-   从 Flink connector 1.2.4 开始，exactly-once 是基于 [Stream Load transaction interface](https://docs.starrocks.io/en-us/latest/loading/Stream_Load_transaction_interface) 重新设计的，该接口由 StarRocks 从 2.4 版本开始提供。与之前基于非事务性 Stream Load 接口的实现相比，新的实现减少了内存使用和检查点开销，从而增强了加载的实时性和稳定性。

-   如果 StarRocks 的版本早于 2.4 或者 Flink connector 的版本早于 1.2.4，sink 会自动选择基于 Stream Load 非事务接口的实现。

- 保证 exactly-once 的配置：

-   `sink.semantic` 的值必须是 `exactly-once`。

-   如果 Flink connector 版本为 1.2.8 及以上，建议指定 `sink.label-prefix` 的值。请注意，标签前缀在 StarRocks 中所有类型的加载中必须是唯一的，例如 Flink 作业、Routine Load 和 Broker Load。

-     如果指定了标签前缀，Flink connector 将使用标签前缀来清理在某些 Flink 失败场景中可能生成的延迟事务，例如当检查点仍在进行时 Flink 作业失败。这些延迟事务通常处于 `PREPARED` 状态，如果您使用 `SHOW PROC '/transactions/<db_id>/running';` 在 StarRocks 中查看它们。当 Flink 作业从检查点恢复时，Flink connector 会根据标签前缀和检查点中的一些信息找到这些延迟事务，并中止它们。由于两阶段提交机制实现了 exactly-once，Flink connector 无法在 Flink 作业退出时中止它们。当 Flink 作业退出时，Flink connector 尚未收到来自 Flink 检查点协调器的通知，是否应将事务包含在成功的检查点中，如果无论如何中止这些事务，则可能会导致数据丢失。您可以在这篇[博文](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)中了解如何在 Flink 中实现端到端的 exactly-once。

-     如果未指定标签前缀，则延迟事务只有在超时后才会被 StarRocks 清理。然而，如果 Flink 作业在事务超时之前频繁失败，则运行的事务数量可能会达到 StarRocks `max_running_txn_num_per_db` 的限制。超时长度由 StarRocks FE 配置 `prepared_transaction_default_timeout_second` 控制，其默认值为 `86400`（1 天）。如果不指定标签前缀，可以设置较小的值，使事务更快过期。

- 如果您确定 Flink 作业在停止或连续故障转移后最终会从检查点或保存点恢复，请相应地调整以下 StarRocks 配置，以避免数据丢失。

-   `prepared_transaction_default_timeout_second`：StarRocks FE 配置，默认值为 `86400`。该配置的值需要大于 Flink 作业的停机时间。否则，成功检查点中包含的延迟事务可能会在重新启动 Flink 作业之前因超时而中止，从而导致数据丢失。

    注意，当您给这个配置设置较大的值时，最好指定 `sink.label-prefix` 的值，这样可以根据标签前缀和检查点中的一些信息来清理延迟事务，而不是因为超时（这可能会导致数据丢失）。

-   `label_keep_max_second` 和 `label_keep_max_num`：StarRocks FE 配置，默认值分别为 `259200` 和 `1000`。详细信息请参见 [FE 配置](../loading/Loading_intro.md#fe-configurations)。`label_keep_max_second` 的值需要大于 Flink 作业的停机时间。否则，Flink connector 无法通过 Flink 的保存点或检查点中保存的事务标签来检查 StarRocks 中事务的状态，并判断这些事务是否已提交，最终可能导致数据丢失。

  这些配置是可变的，可以使用 `ADMIN SET FRONTEND CONFIG` 进行修改：

  ```SQL
    ADMIN SET FRONTEND CONFIG ("prepared_transaction_default_timeout_second" = "3600");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "259200");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_num" = "1000");
  ```

### Flush 策略

Flink 连接器会将数据缓冲在内存中，并通过 Stream Load 批量刷新到 StarRocks。触发刷新的方式在 at-least-once 和 exactly-once 之间有所不同。

对于 at-least-once，当满足以下任一条件时将触发刷新：

- 缓冲行的字节数达到 `sink.buffer-flush.max-bytes` 的限制
- 缓冲行数达到 `sink.buffer-flush.max-rows` 的限制（仅适用于 sink 版本 V1）
- 自上次刷新以来经过的时间达到 `sink.buffer-flush.interval-ms` 的限制
- 触发了检查点

对于 exactly-once，刷新仅在触发检查点时发生。

### 监控加载指标

Flink 连接器提供以下指标来监控加载。

|指标|类型|描述|
|---|---|---|
|totalFlushBytes|计数器|成功刷新的字节数。|
|totalFlushRows|计数器|成功刷新的行数。|
|totalFlushSucceededTimes|计数器|数据成功刷新的次数。|
|totalFlushFailedTimes|计数器|数据刷新失败的次数。|
|totalFilteredRows|计数器|过滤的行数，也包括在 totalFlushRows 中。|

### Flink CDC 同步（支持 schema 变更）

[Flink CDC 3.0](https://github.com/ververica/flink-cdc-connectors/releases) 框架可以用于轻松构建从 CDC 源（如 MySQL 和 Kafka）到 StarRocks 的流式 ELT 管道。该管道可以同步源到 StarRocks 的整个数据库、合并分片表和 schema 变更。

从 v1.2.9 起，StarRocks 的 Flink 连接器作为 [StarRocks Pipeline Connector](https://ververica.github.io/flink-cdc-connectors/master/content/pipelines/starrocks-pipeline.html) 集成到此框架中。StarRocks Pipeline Connector 支持：

- 自动创建数据库和表
- schema 变更同步
- 全量和增量数据同步

快速入门，请参阅 [使用 Flink CDC 3.0 和 StarRocks Pipeline Connector 从 MySQL 到 StarRocks 的流式 ELT](https://ververica.github.io/flink-cdc-connectors/master/content/quickstart/mysql-starrocks-pipeline-tutorial.html)。

## 示例

以下示例展示了如何使用 Flink 连接器通过 Flink SQL 或 Flink DataStream 将数据加载到 StarRocks 表中。

### 准备工作

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

- 下载 Flink 二进制文件 [Flink 1.15.2](https://archive.apache.org/dist/flink/flink-1.15.2/flink-1.15.2-bin-scala_2.12.tgz)，并解压到目录 `flink-1.15.2`。
- 下载[Flink connector 1.2.7](https://repo1.maven.org/maven2/com/starrocks/flink-connector-starrocks/1.2.7_flink-1.15/flink-connector-starrocks-1.2.7_flink-1.15.jar)，并将其放入`flink-1.15.2/lib`目录。
- 执行以下命令启动Flink集群：

  ```shell
  cd flink-1.15.2
  ./bin/start-cluster.sh
  ```

### 使用 Flink SQL 运行

- 运行以下命令启动 Flink SQL 客户端。

  ```shell
  ./bin/sql-client.sh
  ```

- 创建Flink表`score_board`，并通过Flink SQL客户端向表中插入值。注意，如果要将数据加载到StarRocks的主键表中，则必须在Flink DDL中定义主键。对于其他类型的StarRocks表来说，它是可选的。

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

根据输入记录的类型，有多种方法可以实现 Flink DataStream 作业，例如 CSV Java `String`、JSON Java `String` 或自定义 Java 对象。

- 输入记录是 CSV 格式的`String`。有关完整示例，请参阅[LoadCsvRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCsvRecords.java)。

  ```java
  /**
   * 生成 CSV 格式的记录。每条记录有三个由"\t"分隔的值。
   * 这些值将被加载到StarRocks表的`id`、`name`和`score`列中。
   */
  String[] records = new String[]{
          "1\tstarrocks-csv\t100",
          "2\tflink-csv\t100"
  };
  DataStream<String> source = env.fromElements(records);
  
  /**
   * 使用所需的属性配置连接器。
   * 您还需要添加属性"sink.properties.format"和"sink.properties.column_separator"
   * 来告诉连接器输入记录是CSV格式，列分隔符是"\t"。
   * 您也可以在CSV格式的记录中使用其他列分隔符，
   * 但请记得相应地修改"sink.properties.column_separator"。
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
  // 使用选项创建sink。
  SinkFunction<String> starRockSink = StarRocksSink.sink(options);
  source.addSink(starRockSink);
  ```

- 输入记录是 JSON 格式的`String`。有关完整示例，请参阅[LoadJsonRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadJsonRecords.java)。

  ```java
  /**
   * 生成 JSON 格式的记录。
   * 每条记录有三个键值对，对应于StarRocks表中的`id`、`name`和`score`列。
   */
  String[] records = new String[]{
          "{\"id\":1, \"name\":\"starrocks-json\", \"score\":100}",
          "{\"id\":2, \"name\":\"flink-json\", \"score\":100}",
  };
  DataStream<String> source = env.fromElements(records);
  
  /** 
   * 使用所需的属性配置连接器。
   * 您还需要添加属性"sink.properties.format"和"sink.properties.strip_outer_array"
   * 来告诉连接器输入记录是JSON格式并去除最外层的数组结构。
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
  // 使用选项创建sink。
  SinkFunction<String> starRockSink = StarRocksSink.sink(options);
  source.addSink(starRockSink);
  ```

- 输入记录是自定义 Java 对象。有关完整示例，请参阅[LoadCustomJavaRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCustomJavaRecords.java)。

-   在此示例中，输入记录是一个简单的 POJO `RowData`。

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

-   主程序如下：

    ```java
    // 生成使用RowData作为容器的记录。
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
     * Flink连接器将使用Java对象数组(Object[])来表示要加载到StarRocks表中的一行，
     * 每个元素是一个列的值。
     * 您需要定义与StarRocks表匹配的Object[]的模式。
     */
    TableSchema schema = TableSchema.builder()
            .field("id", DataTypes.INT().notNull())
            .field("name", DataTypes.STRING())
            .field("score", DataTypes.INT())
            // 当StarRocks表是主键表时，您必须为主键`id`指定notNull()，例如，DataTypes.INT().notNull()。
            .primaryKey("id")
            .build();
    // 根据模式将RowData转换为Object[]。
    RowDataTransformer transformer = new RowDataTransformer();
    // 使用模式、选项和转换器创建sink。
    SinkFunction<RowData> starRockSink = StarRocksSink.sink(schema, options, transformer);
    source.addSink(starRockSink);
    ```

-   主程序中的`RowDataTransformer`定义如下：

    ```java
    private static class RowDataTransformer implements StarRocksSinkRowBuilder<RowData> {
    
        /**
         * 根据输入的RowData设置对象数组的每个元素。
         * 数组的模式与StarRocks表的模式匹配。
         */
        @Override
        public void accept(Object[] internalRow, RowData rowData) {
            internalRow[0] = rowData.id;
            internalRow[1] = rowData.name;
            internalRow[2] = rowData.score;
            // 当StarRocks表是主键表时，您需要设置最后一个元素以指示数据加载是UPSERT还是DELETE操作。
            internalRow[internalRow.length - 1] = StarRocksSinkOP.UPSERT.ordinal();
        }
    }  
    ```

## 最佳实践

### 将数据加载到主键表

本节将展示如何加载数据到StarRocks主键表中，以实现部分更新和条件更新。
您可以查看[通过加载更改数据](https://docs.starrocks.io/en-us/latest/loading/Load_to_Primary_Key_tables)了解这些功能的介绍。
这些示例使用Flink SQL。

#### 准备工作

在StarRocks中创建数据库`test`并创建主键表`score_board`。

```SQL
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
    `id` INT(11) NOT NULL COMMENT "",
    `name` VARCHAR(65533) NULL DEFAULT "" COMMENT "",
    `score` INT(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`);
```

#### 部分更新

此示例将展示如何仅将数据加载到列`id`和`name`。

1. 在MySQL客户端向StarRocks表`score_board`中插入两行数据。

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

2. 在Flink SQL客户端中创建Flink表`score_board`。

   - 定义仅包含列`id`和`name`的DDL。
   - 将选项`sink.properties.partial_update`设置为`true`，这会告诉Flink连接器执行部分更新。
   - 如果Flink连接器版本`<=`1.2.7，还需要将选项`sink.properties.columns`设置为`id,name,__op`来告诉Flink连接器哪些列需要更新。请注意，您需要在末尾附加字段`__op`。字段`__op`表示数据加载是UPSERT或DELETE操作，其值由连接器自动设置。

   ```SQL
   CREATE TABLE `score_board` (
       `id` INT,
       `name` STRING,
       PRIMARY KEY (id) NOT ENFORCED
   ) WITH (
```
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
    'sink.properties.partial_update' = 'true',
    -- 仅适用于 Flink connector 版本 <= 1.2.7
    'sink.properties.columns' = 'id,name,__op'
);
```

3. 向 Flink 表中插入两行数据。数据行的主键与 StarRocks 表中的行的主键相同，但列 `name` 中的值被修改。

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

您可以看到，只有 `name` 的值发生了变化，`score` 的值没有变化。

#### 条件更新

本示例将展示如何根据列 `score` 的值进行条件更新。仅当新的 `score` 值大于或等于旧值时，`id` 的更新才会生效。

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

2. 创建 Flink 表 `score_board`：

   - 定义包括所有列的 DDL。
   - 将选项 `sink.properties.merge_condition` 设置为 `score` 以告诉连接器使用列 `score` 作为条件。
   - 将选项 `sink.version` 设置为 `V1`，这告诉连接器使用 Stream Load。

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

3. 向 Flink 表中插入两行数据。数据行的主键与 StarRocks 表中的行的主键相同。第一数据行的列 `score` 值较小，第二数据行的列 `score` 值较大。

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

可以看到，只有第二个数据行的值发生变化，第一个数据行的值没有变化。

### 将数据加载到 BITMAP 类型的列中

[`BITMAP`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-statements/data-types/BITMAP) 常用于加速 Count Distinct，例如统计 UV，请参见 [使用 Bitmap 进行精确 Count Distinct](https://docs.starrocks.io/en-us/latest/using_starrocks/Using_bitmap)。这里以 UV 计数为例，展示如何将数据加载到 `BITMAP` 类型的列中。

1. 在 MySQL 客户端中创建 StarRocks 聚合表。

   在数据库 `test` 中，创建聚合表 `page_uv`，其中列 `visit_users` 定义为 `BITMAP` 类型，并配置聚合函数 `BITMAP_UNION`。

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

   Flink 表中的 `visit_user_id` 列是 `BIGINT` 类型，我们希望将该列加载到 StarRocks 表中 `BITMAP` 类型的 `visit_users` 列中。所以在定义 Flink 表的 DDL 时需要注意：
   - 由于 Flink 不支持 `BITMAP`，所以需要定义 `BIGINT` 类型的列 `visit_user_id` 来表示 StarRocks 表中 `BITMAP` 类型的列 `visit_users`。
   - 您需要将选项 `sink.properties.columns` 设置为 `page_id,visit_date,visit_user_id,visit_users=to_bitmap(visit_user_id)`，它告诉连接器 Flink 表和 StarRocks 表之间的列映射。另外还需要使用 [`to_bitmap`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-functions/bitmap-functions/to_bitmap) 函数告诉连接器将 `BIGINT` 类型的数据转换为 `BITMAP` 类型。

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

4. 从 StarRocks 表中计算页面 UV。

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

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md) 可用于近似非重复计数，请参阅 [使用 HLL 进行近似非重复计数](../using_starrocks/Using_HLL.md)。

这里我们以 UV 的计数为例，展示如何将数据加载到 `HLL` 类型的列中。

1. 创建 StarRocks 聚合表

   在数据库 `test` 中，创建聚合表 `hll_uv`，其中列 `visit_users` 定义为 `HLL` 类型，并配置聚合函数 `HLL_UNION`。

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

   Flink 表中的 `visit_user_id` 列是 `BIGINT` 类型，我们希望将该列加载到 StarRocks 表中 `HLL` 类型的 `visit_users` 列中。所以在定义 Flink 表的 DDL 时需要注意：
   - 由于 Flink 不支持 `BITMAP`，所以需要定义 `BIGINT` 类型的列 `visit_user_id` 来表示 StarRocks 表中 `HLL` 类型的列 `visit_users`。
   - 您需要将选项 `sink.properties.columns` 设置为 `page_id,visit_date,visit_user_id,visit_users=hll_hash(visit_user_id)`，它告诉连接器 Flink 表和 StarRocks 表之间的列映射。另外还需要使用 [`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md) 函数告诉连接器将 `BIGINT` 类型的数据转换为 `HLL` 类型。

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

3. 在 Flink SQL 客户端中将数据加载到 Flink 表中。

```SQL
INSERT INTO `hll_uv` VALUES
   (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
   (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
   (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
```

4. 从 MySQL 客户端中的 StarRocks 表计算页面 UV。

```SQL
mysql> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
+---------+-----------------------------+
| page_id | count(DISTINCT visit_users) |
+---------+-----------------------------+
|       3 |                           2 |
|       4 |                           1 |
+---------+-----------------------------+
2 rows in set (0.04 sec)
```
```SQL
INSERT INTO `hll_uv` VALUES
   (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
   (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
   (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
```

4. 从 MySQL 客户端查询 StarRocks 表，计算页面 UV。

   ```SQL
   mysql> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
   +---------+-----------------------------+
   | page_id | count(DISTINCT visit_users) |
   +---------+-----------------------------+
   |       3 |                           2 |
   |       4 |                           1 |
   +---------+-----------------------------+
   2 行记录集 (0.04 秒)
   ```