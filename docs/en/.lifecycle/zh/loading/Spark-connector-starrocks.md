---
displayed_sidebar: English
---

# 使用 Spark 连接器加载数据（推荐）

StarRocks 提供了一款自研的连接器，名为 StarRocks Connector for Apache Spark™（简称 Spark Connector），可帮助您使用 Spark 将数据加载到 StarRocks 表中。其基本原理是累积数据，然后通过 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 一次性将数据加载到 StarRocks 中。Spark 连接器是基于 Spark DataSource V2 实现的，可通过 Spark DataFrames 或 Spark SQL 创建 DataSource，并支持批处理和结构化流模式。

> **注意**
>
> 只有对 StarRocks 表拥有 SELECT 和 INSERT 权限的用户才能将数据加载到该表中。您可以按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中提供的说明向用户授予这些权限。

## 版本要求

| Spark 连接器 | Spark            | StarRocks     | Java | Scala |
| --------------- | ---------------- | ------------- | ---- | ----- |
| 1.1.1 | 3.2、3.3 或 3.4 | 2.5 及更高版本 | 8 | 2.12 |
| 1.1.0           | 3.2、3.3 或 3.4 | 2.5 及更高版本 | 8    | 2.12  |

> **注意**
>
> - 有关 Spark 连接器不同版本之间行为变化的详细信息，请参阅[升级 Spark 连接器](#upgrade-spark-connector)。
> - 从版本 1.1.1 开始，Spark 连接器不再提供 MySQL JDBC 驱动程序，您需要手动将驱动程序导入 Spark 类路径。您可以在 [MySQL 站点](https://dev.mysql.com/downloads/connector/j/) 或 [Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/) 上找到该驱动程序。

## 获取 Spark 连接器

您可以通过以下方式获取 Spark 连接器的 JAR 文件：

- 直接下载已编译的 Spark Connector JAR 文件。
- 将 Spark 连接器作为 Maven 项目的依赖项，然后下载 JAR 文件。
- 自行将 Spark Connector 的源代码编译成 JAR 文件。

Spark 连接器的命名格式为 `starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`。

例如，如果您的环境安装了 Spark 3.2 和 Scala 2.12，并且想要使用 Spark 连接器 1.1.0，则可以使用 `starrocks-spark-connector-3.2_2.12-1.1.0.jar`。

> **注意**
>
> 通常，Spark 连接器的最新版本仅与最近的三个 Spark 版本兼容。

### 下载已编译的 Jar 文件

直接从 [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) 下载相应版本的 Spark 连接器 JAR。

### Maven 依赖

1. 在 Maven 项目的 `pom.xml` 文件中，根据以下格式将 Spark 连接器添加为依赖项。将 `spark_version`、 `scala_version` 和 `connector_version` 替换为相应的版本。

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
    <version>${connector_version}</version>
    </dependency>
    ```

2. 例如，如果您的环境中 Spark 的版本为 3.2，Scala 的版本为 2.12，并且您选择了 Spark 连接器 1.1.0，则需要添加以下依赖项：

    ```xml
    <dependency>
    <groupId>com.starrocks</groupId>
    <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
    <version>1.1.0</version>
    </dependency>
    ```

### 自行编译

1. 下载 [Spark 连接器包](https://github.com/StarRocks/starrocks-connector-for-apache-spark)。
2. 执行以下命令，将 Spark 连接器的源代码编译成 JAR 文件。请注意，`spark_version` 需替换为相应的 Spark 版本。

      ```bash
      sh build.sh <spark_version>
      ```

   例如，如果您的环境中 Spark 的版本为 3.2，则需要执行以下命令：

      ```bash
      sh build.sh 3.2
      ```

3. 进入 `target/` 目录，找到编译时生成的 Spark 连接器 JAR 文件，例如 `starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar`。

> **注意**
>
> 未正式发布的 Spark 连接器的名称包含 `SNAPSHOT` 后缀。

## 参数

| 参数                                      | 必填 | 默认值 | 描述                                                  |
| ---------------------------------------------- | -------- | ------------- | ------------------------------------------------------------ |
| starrocks.fe.http.url                          | 是      | 无          | StarRocks 集群中 FE 的 HTTP URL。您可以指定多个 URL，用逗号（,）分隔。格式：`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>`。从版本 1.1.1 开始，还可以在 URL 前添加 `http://` 前缀，例如 `http://<fe_host1>:<fe_http_port1>,http://<fe_host2>:<fe_http_port2>`。|
| starrocks.fe.jdbc.url                          | 是      | 无          | 用于连接 FE 的 MySQL 服务器的地址。格式：`jdbc:mysql://<fe_host>:<fe_query_port>`。 |
| starrocks.table.identifier                     | 是      | 无          | StarRocks 表的名称。格式：`<database_name>.<table_name>`。 |
| starrocks.user                                 | 是      | 无          | 您的 StarRocks 集群账号的用户名。用户需要对 StarRocks 表拥有 [SELECT 和 INSERT 权限](../sql-reference/sql-statements/account-management/GRANT.md)。            |
| starrocks.password                             | 是      | 无          | 您的 StarRocks 集群账号密码。              |
| starrocks.write.label.prefix                   | 否       | spark-        | Stream Load 使用的标签前缀。                        |
| starrocks.write.enable.transaction-stream-load | 否 | TRUE | 是否使用 [Stream Load 事务接口](../loading/Stream_Load_transaction_interface.md) 加载数据。需要 StarRocks v2.5 及以上版本。此功能可以在事务中加载更多数据，同时减少内存使用量，并提高性能。<br/> **注意：**自 1.1.1 版本起，由于 `starrocks.write.max.retries` Stream Load 事务接口不支持重试，因此该参数仅在 `starrocks.write.max.retries` 的值为非正时生效。|
| starrocks.write.buffer.size                    | 否       | 104857600     | 单次发送到 StarRocks 之前，内存中可以累积的最大数据量。将此参数设置为较大的值可以提高加载性能，但可能会增加加载延迟。 |
| starrocks.write.buffer.rows | 否 | Integer.MAX_VALUE | 1.1.1 版开始支持。单次发送到 StarRocks 之前，内存中可以累积的最大行数。 |
| starrocks.write.flush.interval.ms              | 否       | 300000        | 向 StarRocks 发送数据的时间间隔。该参数用于控制加载延迟。 |
| starrocks.write.max.retries                    | 否       | 3             | 1.1.1 版开始支持。如果加载失败，连接器会重试对同一批数据执行流加载的次数。 <br/> **注意：** 因为 Stream Load 事务接口不支持重试。如果此参数为正值，则连接器将始终使用流加载接口并忽略 `starrocks.write.enable.transaction-stream-load` 的值。|
| starrocks.write.retry.interval.ms              | 否       | 10000         | 1.1.1 版开始支持。如果加载失败，则对同一批数据重试流加载的时间间隔。|
| starrocks.columns                              | 否       | 无          | 需要加载数据的 StarRocks 表列。您可以指定多个列，用逗号（,）分隔，例如 `"col0,col1,col2"`。 |
| starrocks.column.types                         | 否       | 无          | 1.1.1 版开始支持。自定义 Spark 的列数据类型，而不是使用从 StarRocks 表推断的默认值和[默认映射](#data-type-mapping-between-spark-and-starrocks)。参数值是 DDL 格式的 schema，与 Spark [StructType#toDDL](https://github.com/apache/spark/blob/master/sql/api/src/main/scala/org/apache/spark/sql/types/StructType.scala#L449) 的输出相同，例如 `col0 INT, col1 STRING, col2 BIGINT`。请注意，您只需要指定需要自定义的列。一个用例是将数据加载到 [BITMAP](#load-data-into-columns-of-bitmap-type) 或 [HLL](#load-data-into-columns-of-hll-type) 类型的列中。|
| starrocks.write.properties.*                   | 否       | 无          | 用于控制流加载行为的参数。 例如，该参数 `starrocks.write.properties.format` 指定要加载的数据的格式，例如 CSV 或 JSON。有关支持的参数及其说明的列表，请参阅 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。 |
| starrocks.write.properties.format              | 否       | CSV           | Spark 连接器在将数据发送到 StarRocks 之前对每批数据进行转换所依据的文件格式。取值范围：CSV 和 JSON。 |
| starrocks.write.properties.row_delimiter       | 否       | \n            | CSV 格式数据的行分隔符。                    |
| starrocks.write.properties.column_separator    | 否       | \t            | CSV 格式数据的列分隔符。                 |
| starrocks.write.num.partitions                 | 否       | 无          | Spark 可以并行写入数据的分区数。当数据量较小时，可以减少分区数，降低加载并发和频率。此参数的默认值由 Spark 确定。但是，此方法可能会导致 Spark Shuffle 成本。 |
| starrocks.write.partition.columns              | 否       | 无          | Spark 中的分区列。该参数仅在指定时生效 `starrocks.write.num.partitions` 。如果未指定此参数，则所有正在写入的列都用于分区。 |
| starrocks.timezone | 否 | JVM 的默认时区 | 1.1.1 版开始支持。用于将 Spark 的 `TimestampType` 转换为 StarRocks 的 `DATETIME` 的时区。默认值是 JVM 返回的时区 `ZoneId#systemDefault()`。格式可以是时区名称（如 `Asia/Shanghai`），也可以是时区偏移量（如 `+08:00`）。 |

## Spark 和 StarRocks 的数据类型映射关系

- 默认数据类型映射如下：

  | Spark 数据类型 | StarRocks 数据类型                                          |
  | --------------- | ------------------------------------------------------------ |
  | BooleanType     | BOOLEAN                                                      |
  | ByteType        | TINYINT                                                      |
  | ShortType       | SMALLINT                                                     |
  | IntegerType     | INT                                                          |
  | LongType        | BIGINT                                                       |
  | StringType      | LARGEINT                                                     |
  | FloatType       | FLOAT                                                        |
  | DoubleType      | DOUBLE                                                       |
  | DecimalType     | DECIMAL                                                      |
  | StringType      | CHAR                                                         |
  | StringType      | VARCHAR                                                      |
  | StringType      | STRING                                                       |
  | DateType        | DATE                                                         |
  | TimestampType   | DATETIME                                                     |
  | ARRAY类型       | ARRAY <br /> **注意：** <br /> **自版本1.1.1开始支持**。有关详细步骤，请参阅[将数据加载到ARRAY类型的列](#load-data-into-columns-of-array-type)。 |

- 您还可以自定义数据类型映射。

  例如，StarRocks表包含BITMAP和HLL列，但Spark不支持这两种数据类型。您需要在Spark中自定义相应的数据类型。有关详细步骤，请参阅将数据加载到[BITMAP](#load-data-into-columns-of-bitmap-type)和[HLL](#load-data-into-columns-of-hll-type)列中。 **BITMAP和HLL自1.1.1版本开始支持**。

## 升级Spark连接器

### 从1.1.0升级到1.1.1

- 从1.1.1开始，Spark连接器不提供`mysql-connector-java`，这是MySQL的官方JDBC驱动程序，因为`mysql-connector-java`使用GPL许可证的限制。
  但是，Spark连接器仍然需要MySQL JDBC驱动程序来连接StarRocks以获取表元数据，因此需要手动将驱动添加到Spark类路径中。您可以在
  [MySQL站点](https://dev.mysql.com/downloads/connector/j/)或[Maven Central上的驱动程序](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)上找到驱动程序。
- 从1.1.1开始，连接器默认使用Stream Load接口，而不是1.1.0版本中的Stream Load事务接口。如果您仍想使用Stream Load事务接口，则
  可以将选项设置为`starrocks.write.max.retries` `0`。有关详细信息，请参阅`starrocks.write.enable.transaction-stream-load`和`starrocks.write.max.retries`。

## 例子

以下示例展示了如何使用Spark连接器将数据加载到StarRocks表中，其中包含Spark DataFrames或Spark SQL。Spark DataFrames支持批处理和结构化流处理模式。

有关更多示例，请参阅[Spark连接器示例](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/main/src/test/java/com/starrocks/connector/spark/examples)。

### 准备

#### 创建StarRocks表

创建数据库`test`并创建主键表`score_board`。

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

#### 设置Spark环境

请注意，以下示例在Spark 3.2.4中运行，并使用`spark-shell`、`pyspark`和`spark-sql`。在运行示例之前，请确保将Spark连接器JAR文件放在目录`$SPARK_HOME/jars`中。

### 使用Spark DataFrames加载数据

以下两个示例说明了如何使用Spark DataFrames、批处理或结构化流处理模式加载数据。

#### 批处理

在内存中构造数据，并将数据加载到StarRocks表中。

1. 您可以使用Scala或Python编写Spark应用程序。

  对于Scala，请在`spark-shell`中运行以下代码片段：

  ```Scala
  // 1. 从序列创建DataFrame。
  val data = Seq((1, "starrocks", 100), (2, "spark", 100))
  val df = data.toDF("id", "name", "score")

  // 2. 通过将格式配置为"starrocks"和以下选项来写入StarRocks。
  // 您需要根据自己的环境修改选项。
  df.write.format("starrocks")
      .option("starrocks.fe.http.url", "127.0.0.1:8030")
      .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
      .option("starrocks.table.identifier", "test.score_board")
      .option("starrocks.user", "root")
      .option("starrocks.password", "")
      .mode("append")
      .save()
  ```

  对于Python，请在`pyspark`中运行以下代码片段：

   ```python
   from pyspark.sql import SparkSession
   
   spark = SparkSession \
        .builder \
        .appName("StarRocks Example") \
        .getOrCreate()
   
    # 1. 从序列创建DataFrame。
    data = [(1, "starrocks", 100), (2, "spark", 100)]
    df = spark.sparkContext.parallelize(data) \
            .toDF(["id", "name", "score"])

    # 2. 通过将格式配置为"starrocks"和以下选项来写入StarRocks。
    # 您需要根据自己的环境修改选项。
    df.write.format("starrocks") \
        .option("starrocks.fe.http.url", "127.0.0.1:8030") \
        .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030") \
        .option("starrocks.table.identifier", "test.score_board") \
        .option("starrocks.user", "root") \
        .option("starrocks.password", "") \
        .mode("append") \
        .save()
    ```

2. 查询StarRocks表中的数据。

    ```sql
    MySQL [test]> SELECT * FROM `score_board`;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | spark     |   100 |
    +------+-----------+-------+
    2 rows in set (0.00 sec)
    ```

#### 结构化流处理

构建CSV文件中的数据流式读取，并将数据加载到StarRocks表中。

1. 在目录`csv-data`中，创建一个包含以下数据的CSV文件`test.csv`：

    ```csv
    3,starrocks,100
    4,spark,100
    ```

2. 您可以使用Scala或Python编写Spark应用程序。

  对于Scala，请在`spark-shell`中运行以下代码片段：

    ```Scala
    import org.apache.spark.sql.types.StructType

    // 1. 从CSV创建DataFrame。
    val schema = (new StructType()
            .add("id", "integer")
            .add("name", "string")
            .add("score", "integer")
        )
    val df = (spark.readStream
            .option("sep", ",")
            .schema(schema)
            .format("csv") 
            // 用目录“csv-data”的路径替换它。
            .load("/path/to/csv-data")
        )
    
    // 2. 通过将格式配置为"starrocks"和以下选项来写入StarRocks。
    // 您需要根据自己的环境修改选项。
    val query = (df.writeStream.format("starrocks")
            .option("starrocks.fe.http.url", "127.0.0.1:8030")
            .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
            .option("starrocks.table.identifier", "test.score_board")
            .option("starrocks.user", "root")
            .option("starrocks.password", "")
            // 用您的检查点目录替换它
            .option("checkpointLocation", "/path/to/checkpoint")
            .outputMode("append")
            .start()
        )
    ```

  对于Python，请在`pyspark`中运行以下代码片段：
   
   ```python
   from pyspark.sql import SparkSession
   from pyspark.sql.types import IntegerType, StringType, StructType, StructField
   
   spark = SparkSession \
        .builder \
        .appName("StarRocks SS Example") \
        .getOrCreate()
   
    # 1. 从CSV创建DataFrame。
    schema = StructType([
            StructField("id", IntegerType()),
            StructField("name", StringType()),
            StructField("score", IntegerType())
        ])
    df = (
        spark.readStream
        .option("sep", ",")
        .schema(schema)
        .format("csv")
        // 用目录“csv-data”的路径替换它。
        .load("/path/to/csv-data")
    )

    # 2. 通过将格式配置为"starrocks"和以下选项来写入StarRocks。
    # 您需要根据自己的环境修改选项。
    query = (
        df.writeStream.format("starrocks")
        .option("starrocks.fe.http.url", "127.0.0.1:8030")
        .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
        .option("starrocks.table.identifier", "test.score_board")
        .option("starrocks.user", "root")
        .option("starrocks.password", "")
        // 用您的检查点目录替换它
        .option("checkpointLocation", "/path/to/checkpoint")
        .outputMode("append")
        .start()
    )
    ```

3. 查询StarRocks表中的数据。

    ```SQL
    MySQL [test]> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    4 | spark     |   100 |
    |    3 | starrocks |   100 |
    +------+-----------+-------+
    2 rows in set (0.67 sec)
    ```

### 使用Spark SQL加载数据

以下示例说明了如何使用Spark SQL CLI`INSERT INTO`语句[通过Spark SQL加载数据](https://spark.apache.org/docs/latest/sql-distributed-sql-engine-spark-sql-cli.html)。

1. 在`spark-sql`：

    ```SQL
    -- 1. 通过将数据源配置为`starrocks`和以下选项来创建表。
    -- 您需要根据自己的环境修改选项。
    CREATE TABLE `score_board`
    USING starrocks
    OPTIONS(
    "starrocks.fe.http.url"="127.0.0.1:8030",
    "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
    "starrocks.table.identifier"="test.score_board",
    "starrocks.user"="root",
    "starrocks.password"=""
    );

    -- 2. 向表中插入两行数据。
    INSERT INTO `score_board` VALUES (5, "starrocks", 100), (6, "spark", 100);
    ```

2. 查询StarRocks表中的数据。

    ```SQL
    MySQL [test]> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    6 | spark     |   100 |
    |    5 | starrocks |   100 |
    +------+-----------+-------+
    2 rows in set (0.00 sec)
    ```

## 最佳实践

### 将数据加载到主键表

本节将展示如何将数据加载到StarRocks主键表中，以实现部分更新和条件更新。
您可以查看[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)了解这些功能的详细介绍。
这些示例使用Spark SQL。

#### 准备

在StarRocks中创建`test`数据库并在StarRocks中创建主键表`score_board`。

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

此示例将展示如何仅通过加载更新列中的数据`name`：

1. 在MySQL客户端的StarRocks表中插入初始数据。

   ```sql
   mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'spark', 100);

   mysql> select * from score_board;
   +------+-----------+-------+
   | id   | name      | score |
   +------+-----------+-------+
   |    1 | starrocks |   100 |
   |    2 | spark     |   100 |
   +------+-----------+-------+
   2 rows in set (0.02 sec)
   ```

2. 在Spark SQL客户端中创建Spark`score_board`表。

   - 设置`starrocks.write.properties.partial_update`指示连接器执行部分更新`true`的选项。

   - 将选项设置为 `starrocks.columns` `"id,name"` 以告诉连接器要写入哪些列。

   ```SQL
   CREATE TABLE `score_board`
   USING starrocks
   OPTIONS(
       "starrocks.fe.http.url"="127.0.0.1:8030",
       "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
       "starrocks.table.identifier"="test.score_board",
       "starrocks.user"="root",
       "starrocks.password"="",
       "starrocks.write.properties.partial_update"="true",
       "starrocks.columns"="id,name"
    );
   ```

3. 在 Spark SQL 客户端的表中插入数据，并且仅更新列 `name`。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'spark-update');
   ```

4. 在 MySQL 客户端中查询 StarRocks 表。

   您可以看到，只有 `name` 的值发生变化，而 `score` 的值不会改变。

   ```SQL
   mysql> select * from score_board;
   +------+------------------+-------+
   | id   | name             | score |
   +------+------------------+-------+
   |    1 | starrocks-update |   100 |
   |    2 | spark-update     |   100 |
   +------+------------------+-------+
   2 rows in set (0.02 sec)
   ```

#### 有条件的更新

此示例将展示如何根据列 `score` 的值进行有条件的更新。仅当新的 `score` 值大于或等于旧值时，对 `id` 进行更新才会生效。

1. 在 MySQL 客户端的 StarRocks 表中插入初始数据。

    ```SQL
    mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'spark', 100);

    mysql> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | spark     |   100 |
    +------+-----------+-------+
    2 rows in set (0.02 sec)
    ```

2. 通过以下方式创建 Spark 表 `score_board`。

   - 设置 `starrocks.write.properties.merge_condition` 为 `score`，告诉连接器使用列 `score` 作为条件。
   - 确保 Spark 连接器使用 Stream Load 接口加载数据，而不是 Stream Load 事务接口，因为后者不支持此功能。

   ```SQL
   CREATE TABLE `score_board`
   USING starrocks
   OPTIONS(
       "starrocks.fe.http.url"="127.0.0.1:8030",
       "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
       "starrocks.table.identifier"="test.score_board",
       "starrocks.user"="root",
       "starrocks.password"="",
       "starrocks.write.properties.merge_condition"="score"
    );
   ```

3. 在 Spark SQL 客户端中插入数据到表中，将 `id` 为 1 的行更新为较小的分数值，将 `id` 为 2 的行更新为较大的分数值。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'spark-update', 101);
   ```

4. 在 MySQL 客户端中查询 StarRocks 表。

   您可以看到，只有 `id` 为 2 的行发生变化，而 `id` 为 1 的行没有变化。

   ```SQL
   mysql> select * from score_board;
   +------+--------------+-------+
   | id   | name         | score |
   +------+--------------+-------+
   |    1 | starrocks    |   100 |
   |    2 | spark-update |   101 |
   +------+--------------+-------+
   2 rows in set (0.03 sec)
   ```

### 将数据加载到 BITMAP 类型的列中

[`BITMAP`](../sql-reference/sql-statements/data-types/BITMAP.md) 通常用于加速准确计数，例如对 UV 进行计数，请参阅 [使用位图进行准确的重复计数](../using_starrocks/Using_bitmap.md)。
这里我们以 UV 的计数为例，展示如何将数据加载到 `BITMAP` 类型的列中。 **`BITMAP` 自 1.1.1 版起受支持**。

1. 创建 StarRocks 聚合表。

   在数据库 `test` 中，创建一个聚合表 `page_uv`，其中列 `visit_users` 被定义为 `BITMAP` 类型，并配置为使用聚合函数 `BITMAP_UNION`。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT 'page ID',
      `visit_date` datetime NOT NULL COMMENT 'access time',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'user ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. 创建 Spark 表。

    Spark 表的 schema 是从 StarRocks 表中推断出来的，而 Spark 不支持 `BITMAP` 类型。因此，您需要通过配置选项 `"starrocks.column.types"="visit_users BIGINT"` 自定义 Spark 中相应列的数据类型，例如 `BIGINT`。在使用流加载接口加载数据时，连接器会使用 [`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 函数将 `BIGINT` 类型的数据转换为 `BITMAP` 类型。

    在 `spark-sql` 中运行以下 DDL：

    ```SQL
    CREATE TABLE `page_uv`
    USING starrocks
    OPTIONS(
       "starrocks.fe.http.url"="127.0.0.1:8030",
       "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
       "starrocks.table.identifier"="test.page_uv",
       "starrocks.user"="root",
       "starrocks.password"="",
       "starrocks.column.types"="visit_users BIGINT"
    );
    ```

3. 将数据加载到 StarRocks 表中。

    在 `spark-sql` 中运行以下 DML：

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. 根据 StarRocks 表计算页面 UV。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `page_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       2 |                           1 |
    |       1 |                           3 |
    +---------+-----------------------------+
    2 rows in set (0.01 sec)
    ```

> **注意：**
>
> 连接器使用 [`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)
> 函数将 Spark 中的 `TINYINT`、`SMALLINT`、`INTEGER` 和 `BIGINT` 类型的数据转换为 StarRocks 中的 `BITMAP` 类型，并对其他 Spark 数据类型使用 [`bitmap_hash`](../sql-reference/sql-functions/bitmap-functions/bitmap_hash.md) 函数。

### 将数据加载到 HLL 类型的列中

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md) 可用于近似计数非重复性，请参阅 [使用 HLL 进行近似非重复计数](../using_starrocks/Using_HLL.md)。

这里我们以 UV 的计数为例，展示如何将数据加载到 `HLL` 类型的列中。  **`HLL` 自 1.1.1 版起受支持**。

1. 创建 StarRocks 聚合表。

   在数据库 `test` 中，创建一个聚合表 `hll_uv`，其中列 `visit_users` 被定义为 `HLL` 类型，并配置为使用聚合函数 `HLL_UNION`。

    ```SQL
    CREATE TABLE `hll_uv` (
    `page_id` INT NOT NULL COMMENT 'page ID',
    `visit_date` datetime NOT NULL COMMENT 'access time',
    `visit_users` HLL HLL_UNION NOT NULL COMMENT 'user ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. 创建 Spark 表。

   Spark 表的 schema 是从 StarRocks 表中推断出来的，而 Spark 不支持 `HLL` 类型。因此，您需要通过配置选项 `"starrocks.column.types"="visit_users BIGINT"` 自定义 Spark 中相应列的数据类型，例如 `BIGINT`。在使用流加载接口加载数据时，连接器会使用 [`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md) 函数将 `BIGINT` 类型的数据转换为 `HLL` 类型。

    在 `spark-sql` 中运行以下 DDL：

    ```SQL
    CREATE TABLE `hll_uv`
    USING starrocks
    OPTIONS(
       "starrocks.fe.http.url"="127.0.0.1:8030",
       "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
       "starrocks.table.identifier"="test.hll_uv",
       "starrocks.user"="root",
       "starrocks.password"="",
       "starrocks.column.types"="visit_users BIGINT"
    );
    ```

3. 将数据加载到 StarRocks 表中。

    在 `spark-sql` 中运行以下 DML：

    ```SQL
    INSERT INTO `hll_uv` VALUES
       (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
       (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
       (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. 根据 StarRocks 表计算页面 UV。

    ```SQL
    MySQL [test]> SELECT `page_id`, COUNT(DISTINCT `visit_users`) FROM `hll_uv` GROUP BY `page_id`;
    +---------+-----------------------------+
    | page_id | count(DISTINCT visit_users) |
    +---------+-----------------------------+
    |       4 |                           1 |
    |       3 |                           2 |
    +---------+-----------------------------+
    2 rows in set (0.01 sec)
    ```

### 将数据加载到 ARRAY 类型的列中

下面的示例说明如何将数据加载到 [`ARRAY`](../sql-reference/sql-statements/data-types/Array.md) 类型的列中。

1. 创建 StarRocks 表。

   在数据库 `test` 中，创建一个包含一列 `id` 和两列 `ARRAY` 的主键表 `array_tbl`。

   ```SQL
   CREATE TABLE `array_tbl` (
       `id` INT NOT NULL,
       `a0` ARRAY<STRING>,
       `a1` ARRAY<ARRAY<INT>>
    )
    ENGINE=OLAP
    PRIMARY KEY(`id`)
    DISTRIBUTED BY HASH(`id`)
    ;
   ```

2. 向 StarRocks 写入数据。

   由于某些版本的 StarRocks 没有提供 `ARRAY` 列的元数据，连接器无法推断出该列对应的 Spark 数据类型。但是，您可以在选项中显式指定列的相应 Spark 数据类型，例如 `a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>`。

   在 `spark-shell` 中运行以下代码：

   ```scala
    val data = Seq(
       |  (1, Seq("hello", "starrocks"), Seq(Seq(1, 2), Seq(3, 4))),
       |  (2, Seq("hello", "spark"), Seq(Seq(5, 6, 7), Seq(8, 9, 10))
       | )
    val df = data.toDF("id", "a0", "a1")
    df.write
         .format("starrocks")
         .option("starrocks.fe.http.url", "127.0.0.1:8030")
         .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")

         .option("starrocks.table.identifier", "test.array_tbl")
         .option("starrocks.user", "root")
         .option("starrocks.password", "")
         .option("starrocks.column.types", "a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>")
         .mode("append")
         .save()
    ```

3. 查询 StarRocks 表中的数据。

   ```SQL
   MySQL [test]> SELECT * FROM `array_tbl`;
   +------+-----------------------+--------------------+
   | id   | a0                    | a1                 |
   +------+-----------------------+--------------------+
   |    1 | ["hello","starrocks"] | [[1,2],[3,4]]      |
   |    2 | ["hello","spark"]     | [[5,6,7],[8,9,10]] |
   +------+-----------------------+--------------------+
   2 行记录 (0.01 秒)
   ```