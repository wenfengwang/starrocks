---
displayed_sidebar: "Chinese"
---

# Continuous import from Apache Flink®

StarRocks provides an Apache Flink® connector, which can import data to StarRocks tables through Flink.

The basic principle is that the Flink connector accumulates small batches of data in memory, and then imports them to StarRocks in one go via [Stream Load](./StreamLoad.md).

The Flink Connector supports DataStream API, Table API & SQL, and Python API.

Compared to the [flink-connector-jdbc](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/jdbc/) provided by Flink, the Flink connector provided by StarRocks has better performance and stability.

> **Note**
>
> To import data to StarRocks using the Flink connector, SELECT and INSERT permissions are required for the target table. If your user account does not have these permissions, please refer to [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) to grant permissions to the user.

## Version Requirement

| Connector | Flink       | StarRocks  | Java | Scala      |
| --------- | ----------- | ---------- | ---- | ---------- |
| 1.2.8     | 1.13 ~ 1.17 | 2.1 and above | 8    | 2.11, 2.12 |
| 1.2.7     | 1.11 ~ 1.15 | 2.1 and above | 8    | 2.11, 2.12 |

## Obtain Flink connector

You can obtain the Flink connector JAR file in the following ways:

- Directly download the compiled JAR file.
- Add the Flink connector as a dependency in the pom file of the Maven project and download it as a dependency.
- Manually compile the source code into a JAR file.

The naming format for the Flink connector JAR file is as follows:

- For Flink versions 1.15 and later, the naming format of the Flink connector is `flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`. For example, if you have installed Flink 1.15 and want to use Flink connector version 1.2.7, you can use `flink-connector-starrocks-1.2.7_flink-1.15.jar`.
- For Flink versions before 1.15, the naming format of the Flink connector is `flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`. For example, if you have installed Flink 1.14 and Scala 2.12 and want to use Flink connector version 1.2.7, you can use `flink-connector-starrocks-1.2.7_flink-1.14_2.12.jar`.

> **Note**
>
> Generally, the latest version of the Flink connector only maintains the latest 3 versions of Flink.

### Direct Download

You can obtain different versions of the Flink connector JAR files from the [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks).

### Maven Dependency

In the `pom.xml` file of the Maven project, add the Flink connector as a dependency according to the following format. Replace `flink_version`, `scala_version`, and `connector_version` with the corresponding versions.

- For Flink versions 1.15 and later

    ```XML
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}</version>
    </dependency>
    ```

- For Flink versions before 1.15

    ```XML
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}_${scala_version}</version>
    </dependency>
    ```

### Manual Compilation

1. Download the [Flink connector code](https://github.com/StarRocks/starrocks-connector-for-apache-flink).
2. Execute the following command to compile the source code of the Flink connector into a JAR file. Note to replace `flink_version` with the corresponding Flink version.

    ```Bash
    sh build.sh <flink_version>
    ```

    For example, if the Flink version in your environment is 1.15, you need to execute the following command:

    ```Bash
    sh build.sh 1.15
    ```

3. Go to the `target/` directory and find the compiled Flink connector JAR file, for example, `flink-connector-starrocks-1.2.7_flink-1.15-SNAPSHOT.jar`, which is generated during the compilation process.

    > **Note**:
    >
    > The name of the unreleased Flink connector includes the `SNAPSHOT` suffix.

## Parameter Description

| Parameter                         | Mandatory | Default Value | Description                                                  |
| --------------------------------- | --------- | ------------- | ------------------------------------------------------------ |
| connector                         | Yes       | NONE          | Set to `starrocks` fixedly.                                  |
| jdbc-url                          | Yes       | NONE          | Used to access the MySQL server on the FE node. Multiple addresses are separated by a comma (,). Format: `jdbc:mysql://<fe_host1>:<fe_query_port1>,<fe_host2>:<fe_query_port2>`. |
| load-url                          | Yes       | NONE          | Used to access the HTTP server on the FE node. Multiple addresses are separated by a semicolon (;). Format: `<fe_host1>:<fe_http_port1>;<fe_host2>:<fe_http_port2>`. |
| database-name                     | Yes       | NONE          | StarRocks database name.                                      |
| table-name                        | Yes       | NONE          | StarRocks table name.                                         |
| username                          | Yes       | NONE          | Username of the StarRocks cluster. SELECT and INSERT permissions are required to use the Flink connector to import data to StarRocks. If your user account does not have these permissions, please refer to [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) to grant permissions to the user. |
| password                          | Yes       | NONE          | User password of the StarRocks cluster.                      |
| sink.semantic                     | No        | at-least-once  | The semantic ensured by the sink. Valid values: **at-least-once** and **exactly-once**. |
| sink.version                      | No        | AUTO          | The interface for importing data. This parameter is supported from Flink connector 1.2.4 onwards.<ul><li>V1: Use the [Stream Load](./StreamLoad.md) interface to import data. Before version 1.2.4 of Flink connector, only this mode is supported.</li> <li>V2: Use the [Stream Load Transaction Interface](../loading/Stream_Load_transaction_interface.md) to import data. StarRocks version must be greater than or equal to 2.4. It is recommended to select V2 as it reduces memory usage and provides a more stable exactly-once implementation.</li> <li>AUTO: If the StarRocks version supports the Stream Load Transaction Interface, V2 will be automatically selected, otherwise V1 will be selected.</li></ul> |
| sink.label-prefix                 | No        | NONE          | Specifies the prefix of the label used by Stream Load. If the Flink connector version is 1.2.8 or above and the sink ensures exactly-once semantics, it is recommended to configure the label prefix. For more information, see [exactly once](#exactly-once).                      |
| sink.buffer-flush.max-bytes       | No       | 94371840(90M) | 积攒在内存的数据大小，达到该阈值后数据通过 Stream Load 一次性导入 StarRocks。取值范围：[64MB, 10GB]。将此参数设置为较大的值可以提高导入性能，但可能会增加导入延迟。 该参数只在 `sink.semantic` 为`at-least-once`才会生效。 `sink.semantic` 为 `exactly-once`，则只有 Flink checkpoint 触发时 flush 内存的数据，因此该参数不生效。 |
| sink.buffer-flush.max-rows        | No       | 500000        | 积攒在内存的数据条数，达到该阈值后数据通过 Stream Load 一次性导入 StarRocks。取值范围：[64000, 5000000]。该参数只在 `sink.version` 为 `V1`，`sink.semantic` 为 `at-least-once` 才会生效。 |
| sink.buffer-flush.interval-ms     | No       | 300000        | 数据发送的间隔，用于控制数据写入 StarRocks 的延迟，取值范围：[1000, 3600000]。该参数只在 `sink.semantic` 为 `at-least-once`才会生效。 |
| sink.max-retries                  | No       | 3             | Stream Load 失败后的重试次数。超过该数量上限，则数据导入任务报错。取值范围：[0, 10]。该参数只在 `sink.version` 为 `V1` 才会生效。 |
| sink.connect.timeout-ms           | No       | 1000          | 与 FE 建立 HTTP 连接的超时时间。取值范围：[100, 60000]。     |
| sink.wait-for-continue.timeout-ms | No       | 10000         | 此参数自 Flink connector 1.2.7 开始支持。等待 FE HTTP 100-continue 应答的超时时间。取值范围：[3000, 60000]。 |
| sink.ignore.update-before         | No       | TRUE          | 此参数自 Flink connector 1.2.8 开始支持。将数据导入到主键模型表时，是否忽略来自 Flink 的 UPDATE_BEFORE 记录。如果将此参数设置为 false，则将该记录在主键模型表中视为DELETE 操作。 |
| sink.parallelism                  | No       | NONE          | 写入的并行度。仅适用于Flink SQL。如果未设置， Flink planner 将决定并行度。**在多并行度的场景中，用户需要确保数据按正确顺序写入。** |
| sink.properties.*                 | No       | NONE          | Stream Load 的参数，控制 Stream Load 导入行为。例如 参数 sink.properties.format 表示 Stream Load 所导入的数据格式，如 CSV 或者 JSON。全部参数和解释，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。 |
| sink.properties.format            | No       | csv           | Stream Load 导入时的数据格式。Flink connector 会将内存的数据转换为对应格式，然后通过 Stream Load 导入至 StarRocks。取值为 CSV 或者 JSON。 |
| sink.properties.column_separator  | No       | \t            | CSV 数据的列分隔符。                                         |
| sink.properties.row_delimiter     | No       | \n            | CSV 数据的行分隔符。                                         |
| sink.properties.max_filter_ratio  | No       | 0             | 导入作业的最大容错率，即导入作业能够容忍的因数据质量不合格而过滤掉的数据行所占的最大比例。取值范围：0~1。默认值：0 。详细信息，请参见  [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。 |

## 数据类型映射

| Flink 数据类型                    | StarRocks 数据类型 |
| --------------------------------- | ------------------ |
| BOOLEAN                           | BOOLEAN            |
| TINYINT                           | TINYINT            |
| SMALLINT                          | SMALLINT           |
| INTEGER                           | INTEGER            |
| BIGINT                            | BIGINT             |
| FLOAT                             | FLOAT              |
| DOUBLE                            | DOUBLE             |
| DECIMAL                           | DECIMAL            |
| BINARY                            | INT                |
| CHAR                              | STRING             |
| VARCHAR                           | STRING             |
| STRING                            | STRING             |
| DATE                              | DATE               |
| TIMESTAMP_WITHOUT_TIME_ZONE(N)    | DATETIME           |
| TIMESTAMP_WITH_LOCAL_TIME_ZONE(N) | DATETIME           |
| ARRAY&lt;T&gt;                    | ARRAY&lt;T&gt;     |
| MAP&lt;KT,VT&gt;                  | JSON STRING        |
| ROW&lt;arg T...&gt;               | JSON STRING        |

## 使用说明

### Exactly Once

- 如果您希望 sink 保证 exactly-once 语义，则建议升级 StarRocks 到 2.5 或更高版本，并将 Flink connector 升级到 1.2.4 或更高版本。

  - 自 2.4 版本 StarRocks 开始支持 [Stream Load 事务接口](https://docs.starrocks.io/zh-cn/latest/loading/Stream_Load_transaction_interface)。自 Flink connector 1.2.4 版本起， Sink 基于 Stream Load 事务接口重新设计 exactly-once 的实现，相较于原来基于 Stream Load 非事务接口实现的 exactly-once，降低了内存使用和 checkpoint 耗时，提高了作业的实时性和稳定性。
  - 自 Flink connector 1.2.4 版本起，如果 StarRocks 支持 Stream Load 事务接口，则 Sink 默认使用 Stream Load 事务接口，如果需要使用 Stream Load  非事务接口实现，则需要配置 `sink.version` 为`V1`。
  > **注意**
  >
  > 如果只升级 StarRocks 或 Flink connector，sink 会自动选择  Stream Load  非事务接口实现。

- sink 保证 exactly-once 语义相关配置
  
  - `sink.semantic` 的值必须为 `exactly-once`.
  
  - 如果 Flink connector 版本为 1.2.8 及更高，则建议指定 `sink.label-prefix` 的值。需要注意的是，label 前缀在 StarRocks 的所有类型的导入作业中必须是唯一的，包括 Flink job、Routine Load 和 Broker Load。

    - 如果指定了 label 前缀，Flink connector 将使用 label 前缀清理因为 Flink job 失败而生成的未完成事务，例如在checkpoint 进行过程中 Flink job 失败。如果使用 `SHOW PROC '/transactions/<db_id>/running';` 查看这些事务在 StarRock 的状态，则返回结果会显示事务通常处于 `PREPARED` 状态。当 Flink job 从 checkpoint 恢复时，Flink connector 将根据 label 前缀和 checkpoint 中的信息找到这些未完成的事务，并中止事务。当 Flink job 因某种原因退出时，由于采用了两阶段提交机制来实现 exactly-once语义，Flink connector 无法中止事务。当 Flink 作业退出时，Flink connector 尚未收到来自 Flink checkpoint coordinator 的通知，说明这些事务是否应包含在成功的 checkpoint 中，如果中止这些事务，则可能导致数据丢失。您可以在这篇[文章](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)中了解如何在 Flink 中实现端到端的 exactly-once。
- 如果未指定 label 前缀，则未完成的事务将在超时后由 StarRocks 清理。然而，如果 Flink job 在事务超时之前频繁失败，则运行中的事务数量可能会达到 StarRocks 的 `max_running_txn_num_per_db` 限制。超时长度由 StarRocks FE 配置 `prepared_transaction_default_timeout_second` 控制，默认值为 `86400`（1天）。如果未指定 label 前缀，您可以设置一个较小的值，使事务更快超时。

- 如果您确定 Flink job 将在长时间停止后最终会使用 checkpoint 或 savepoint 恢复，则为避免数据丢失，请调整以下 StarRocks 配置：

    - `prepared_transaction_default_timeout_second`：StarRocks FE 参数，默认值为 `86400`。此参数值需要大于 Flink job 的停止时间。否则，在重新启动 Flink job 之前，可能会因事务超时而中止未完成事务，这些事务可能包含在成功 checkpoint 中的，如果中止，则会导致数据丢失。
  
    请注意，当您设置一个较大的值时，则建议指定 `sink.label-prefix` 的值，则 Flink connector 可以根据 label 前缀和检查点中的一些信息来清理未完成的事务，而不是因事务超时后由 StarRocks 清理（这可能会导致数据丢失）。

    - `label_keep_max_second` 和 `label_keep_max_num`：StarRocks FE 参数，默认值分别为 `259200` 和 `1000`。更多信息，参见[FE 配置](../loading/Loading_intro.md#fe-配置)。`label_keep_max_second` 的值需要大于 Flink job 的停止时间。否则，Flink connector 无法使用保存在 Flink 的 savepoint 或 checkpoint 中的事务 lable 来检查事务在 StarRocks 中的状态，并判断这些事务是否已提交，最终可能导致数据丢失。

    您可以使用 `ADMIN SET FRONTEND CONFIG` 修改上述配置。

    ```SQL
    ADMIN SET FRONTEND CONFIG ("prepared_transaction_default_timeout_second" = "3600");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "259200");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_num" = "1000");
    ```

### Flush 策略

Flink connector 先在内存中 buffer 数据，然后通过 Stream Load 将其一次性 flush 到 StarRocks。在 at-least-once 和 exactly-once 场景中使用不同的方式触发 flush 。

对于 at-least-once，在满足以下任何条件时触发 flush：

- buffer 数据的字节达到限制 `sink.buffer-flush.max-bytes`
- buffer 数据行数达到限制 `sink.buffer-flush.max-rows`。（仅适用于版本 V1）
- 自上次 flush 以来经过的时间达到限制 `sink.buffer-flush.interval-ms`
- 触发了 checkpoint

对于 exactly-once，仅在触发 checkpoint 时触发 flush。

### 监控导入指标

Flink connector 提供以下指标来监控导入情况。

| 指标名称                     | 类型   | 描述                                               |
| ------------------------ | ------ | -------------------------------------------------- |
| totalFlushBytes          | Counter| 成功 flush 的字节。                                 |
| totalFlushRows           | Counter | 成功 flush 的行数。                                   |
| totalFlushSucceededTimes | Counter |  flush 数据的成功次数。                           |
| totalFlushFailedTimes    | Counter | flush 数据的失败次数。                                   |
| totalFilteredRows        | Counter | 已过滤的行数，这些行数也包含在 totalFlushRows 中。 |

## 使用示例

### 准备工作

#### 创建 StarRocks 表

创建数据库 `test`，并创建主键模型表  `score_board`。

```SQL
CREATE DATABASE test;

CREATE TABLE test.score_board(
    id int(11) NOT NULL COMMENT "",
    name varchar(65533) NULL DEFAULT "" COMMENT "",
    score int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(id)
DISTRIBUTED BY HASH(id);
```

#### Flink 环境

- 下载 Flink 二进制文件 [Flink 1.15.2](https://archive.apache.org/dist/flink/flink-1.15.2/flink-1.15.2-bin-scala_2.12.tgz)，并解压到目录 `flink-1.15.2`。
- 下载 [Flink connector 1.2.7](https://repo1.maven.org/maven2/com/starrocks/flink-connector-starrocks/1.2.7_flink-1.15/flink-connector-starrocks-1.2.7_flink-1.15.jar)，并将其放置在目录 `flink-1.15.2/lib` 中。
- 运行以下命令启动 Flink 集群：

    ```Bash
    cd flink-1.15.2
    ./bin/start-cluster.sh
    ```

### 使用 Flink SQL 写入数据

- 运行以下命令以启动 Flink SQL 客户端。

    ```Bash
    ./bin/sql-client.sh
    ```

- 在 Flink SQL 客户端，创建一个表 `score_board`，并且插入数据。 注意，如果您想将数据导入到 StarRocks 主键模型表中，您必须在 Flink 表的 DDL 中定义主键。对于其他类型的 StarRocks 表，这是可选的。

    ```sql
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

### 使用 Flink DataStream 写入数据

根据 input records 的类型，编写对应 Flink DataStream 作业，例如 input records 为 CSV 格式的 Java `String`、JSON 格式的 Java `String` 或自定义的 Java 对象。

- 如果 input records 为 CSV 格式的 `String`，对应的 Flink DataStream 作业的主要代码如下所示，完整代码请参见 [LoadCsvRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCsvRecords.java)。

    ```Java
    /**
     * Generate CSV-format records. Each record has three values separated by "\t". 
     * These values will be loaded to the columns `id`, `name`, and `score` in the StarRocks table.
     */
    String[] records = new String[]{
            "1\tstarrocks-csv\t100",
            "2\tflink-csv\t100"
    };
    DataStream<String> source = env.fromElements(records);
    
    /**
     * Configure the Flink connector with the required properties.
     * You also need to add properties "sink.properties.format" and "sink.properties.column_separator"
     * to tell the Flink connector the input records are CSV-format, and the column separator is "\t".
     * You can also use other column separators in the CSV-format records,
     * but remember to modify the "sink.properties.column_separator" correspondingly.
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
    // Create the sink with the options.
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 如果 input records 为 JSON 格式的 `String`，对应的 Flink DataStream 作业的主要代码如下所示，完整代码请参见[LoadJsonRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadJsonRecords.java)。

    ```Java
    /**
     * 生成 JSON 格式记录。
     * 每条记录有三对键值对，分别对应 StarRocks 表中的列 id、name 和 score。
     */
    String[] records = new String[]{
            "{\"id\":1, \"name\":\"starrocks-json\", \"score\":100}",
            "{\"id\":2, \"name\":\"flink-json\", \"score\":100}",
    };
    DataStream<String> source = env.fromElements(records);
    
    /** 
     * 使用所需属性配置 Flink 连接器。
     * 您还需要添加属性"sink.properties.format"和"sink.properties.strip_outer_array"来告知 Flink 连接器输入记录是 JSON 格式并消除最外层数组结构。 
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
    // 使用选项创建 Sink。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```


- 如果 input records 为自定义的 Java 对象，对应的 Flink DataStream 作业的主要代码如下所示，完整代码请参见[LoadCustomJavaRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCustomJavaRecords.java)。

  - 本示例中，input record 是一个简单的 POJO `RowData`。

    ```Java
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

  - 主要代码如下所示：

    ```Java
    // 生成使用 RowData 作为容器的记录。
    RowData[] records = new RowData[]{
            new RowData(1, "starrocks-rowdata", 100),
            new RowData(2, "flink-rowdata", 100),
        };
    DataStream<RowData> source = env.fromElements(records);
    
    // 使用所需属性配置 Flink 连接器。
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .build();
    
    /**
     * Flink 连接器将使用 Java 对象数组（Object[]）表示要加载到 StarRocks 表中的行，每个元素都是列的值。
     * 您需要定义与 StarRocks 表匹配的 Object[] 的模式。
     */
    TableSchema schema = TableSchema.builder()
            .field("id", DataTypes.INT().notNull())
            .field("name", DataTypes.STRING())
            .field("score", DataTypes.INT())
            // 当 StarRocks 表是主键表时，您必须指定 notNull()，例如，
            // DataTypes.INT().notNull() 用于主键 `id`。
            .primaryKey("id")
            .build();
    // 根据模式将 RowData 转换为 Object[]。
    RowDataTransformer transformer = new RowDataTransformer();
    // 使用模式、选项和转换器创建 Sink。
    SinkFunction<RowData> starRockSink = StarRocksSink.sink(schema, options, transformer);
    source.addSink(starRockSink);
    ```

  - 其中 `RowDataTransformer` 定义如下：

    ```Java
    private static class RowDataTransformer implements StarRocksSinkRowBuilder<RowData> {
    
        /**
         * 根据 input RowData 设置对象数组的每个元素。
         * 数组的模式与 StarRocks 表的模式匹配。
         */
        @Override
        public void accept(Object[] internalRow, RowData rowData) {
            internalRow[0] = rowData.id;
            internalRow[1] = rowData.name;
            internalRow[2] = rowData.score;
            // 当 StarRocks 表是主键表时，您需要设置最后一个元素以指示数据加载是 UPSERT 还是 DELETE 操作。
            internalRow[internalRow.length - 1] = StarRocksSinkOP.UPSERT.ordinal();
        }
    }  
    ```

## 最佳实践

### 导入至主键模型表

本节将展示如何将数据导入到 StarRocks 主键模型表中，以实现部分更新和条件更新。以下示例使用 Flink SQL。 部分更新和条件更新的更多介绍，请参见[通过导入实现数据变更](./Load_to_Primary_Key_tables.md)。

#### 准备工作

在StarRocks中创建一个名为`test`的数据库，并在其中创建一个名为`score_board`的主键模型表。

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

#### 部分更新

本示例展示如何通过导入数据仅更新 StarRocks 表中列 `name`的值。

1. 在 MySQL 客户端向 StarRocks 表 `score_board` 插入两行数据。

      ```sql
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

2. 在 Flink SQL 客户端创建表 `score_board` 。
   - DDL 中仅包含列 `id` 和 `name` 的定义。
   - 将选项 `sink.properties.partial_update` 设置为 `true`，以要求 Flink connector 执行部分更新。
   - 如果 Flink connector 版本小于等于 1.2.7，则还需要将选项 `sink.properties.columns` 设置为`id,name,__op`，以告诉 Flink connector 需要更新的列。请注意，您需要在末尾附加字段 `__op`。字段 `__op` 表示导入是 UPSERT 还是 DELETE 操作，其值由 Flink connector 自动设置。

      ```sql
      CREATE TABLE score_board (
          id INT,
          name STRING,
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
          -- only for Flink connector version <= 1.2.7
          'sink.properties.columns' = 'id,name,__op'
      ); 
      ```

3. 将两行数据插入两行数据到表中。数据行的主键与 StarRocks 表的数据行主键相同，但是 `name` 列的值被修改。

      ```SQL
      INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'flink-update');
      ```

4. 在 MySQL 客户端查询 StarRocks 表。

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

  

您会看到只有 `name` 列的值发生了变化，而 `score` 列的值没有变化。

#### 条件更新

本示例展示如何根据 `score` 列的值进行条件更新。只有导入的数据行中 `score` 列值大于等于 StarRocks 表当前值时，该数据行才会更新。

1. 在 MySQL 客户端中向 StarRocks 表中插入两行数据。


    ```SQL
    mysql> INSERT INTO score_board VALUES (1, 'starrocks', 100), (2, 'flink', 100);
  
    mysql> select * from score_board;
    +------+-----------+-------+
    +------+-----------+-------+

    +------+-----------+-------+
    2 rows in set (0.02 sec)
    ```


2. 在 Flink SQL 客户端按照以下方式创建表`score_board`：
   - DDL 中包括所有列的定义。
   - 将选项  `sink.properties.merge_condition` 设置为 `score`，要求 Flink connector 使用 `score`  列作为更新条件。
   - 将选项 `sink.version` 设置为 `V1` ，要求 Flink connector 使用 Stream Load 接口导入数据。因为只有 Stream Load 接口支持条件更新。

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

3. 在 Flink SQL 客户端插入两行数据到表中。数据行的主键与 StarRocks 表中的行相同。第一行数据 `score` 列中具有较小的值，而第二行数据 `score` 列中具有较大的值。

      ```SQL
      INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'flink-update', 101);
      ```

4. 在 MySQL客户端查询 StarRocks表。

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

    您会注意到仅第二行数据发生了变化，而第一行数据未发生变化。

### 导入至 Bitmap 列

`BITMAP` 常用于加速精确去重计数，例如计算独立访客数（UV），更多信息，请参见[使用 Bitmap 实现精确去重](../using_starrocks/Using_bitmap.md)。


本示例以计算独立访客数（UV）为例，展示如何导入数据至 StarRocks 表 `BITMAP` 列中。

1. 在 MySQL 客户端中创建一个 StarRocks 聚合表。

   在数据库`test`中，创建聚合表 `page_uv`，其中列 `visit_users` 被定义为 `BITMAP` 类型，并配置聚合函数 `BITMAP_UNION`。


      ```SQL
      CREATE TABLE `test`.`page_uv` (
        `page_id` INT NOT NULL COMMENT 'page ID',
        `visit_date` datetime NOT NULL COMMENT 'access time',
        `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'user ID'
      ) ENGINE=OLAP
      AGGREGATE KEY(`page_id`, `visit_date`)
      DISTRIBUTED BY HASH(`page_id`);
      ```

2. 在 Flink SQL 客户端中创建一个表。

   因为表中的 `visit_user_id` 列是`BIGINT`类型，我们希望将此列的数据导入到StarRocks表中的`visit_users`列，该列是`BITMAP`类型。因此，在定义表的 DDL 时，需要注意以下几点：

   - 由于 Flink 不支持 `BITMAP` 类型，您需要将 `visit_user_id` 列定义为`BIGINT`类型，以代表StarRocks表中的 `visit_users` 列。
   - 您需要将选项 `sink.properties.columns` 设置为`page_id,visit_date,user_id,visit_users=to_bitmap(visit_user_id)`，以告诉 Flink connector 如何将该表的列和 StarRocks 表的列进行映射，并且还需要使用 `to_bitmap` 函数，将`BIGINT` 类型 `visit_user_id` 列的数据转换为 `BITMAP`类型。

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


3. 在 Flink SQL 客户端中插入数据至表中。

      ```SQL

      INSERT INTO `page_uv` VALUES

         (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),

         (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),

         (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
         (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
         (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
      ```

4. 在 MySQL 客户端查询 StarRocks 表来计算页面 UV 数。

      ```SQL
      MySQL [test]> SELECT page_id, COUNT(DISTINCT visit_users) FROM page_uv GROUP BY page_id;

      +---------+-----------------------------+

      +---------+-----------------------------+

- 由于 Flink 不支持`HLL`类型，您需要将 `visit_user_id` 列定义为 `BIGINT` 类型，以代表 StarRocks 表中的 `visit_users` 列。
    - 您需要将选项 `sink.properties.columns` 设置为`page_id,visit_date,user_id,visit_users=hll_hash(visit_user_id)`，以告诉 Flink connector 如何将该表的列和 StarRocks 表的列进行映射。还需要使用 `hll_hash` 函数，将 `BIGINT` 类型的 `visit_user_id` 列的数据转换为 `HLL` 类型。

    ```SQL
    CREATE TABLE hll_uv (
        page_id INT,
        visit_date TIMESTAMP,
        visit_user_id BIGINT
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

3. 在 Flink SQL 客户端中插入数据至表中。

    ```SQL
    INSERT INTO hll_uv VALUES
    (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
    (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
    (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. 在 MySQL 客户端查询 StarRocks 表来计算页面 UV 数。

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