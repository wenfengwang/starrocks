```
---
displayed_sidebar: "Chinese"
---

# 使用Spark连接器加载数据（推荐）

StarRocks提供了一种自研的名为StarRocks Connector for Apache Spark™（简称Spark连接器）的连接器，帮助您通过Spark将数据加载到StarRocks表中。其基本原则是累积数据，然后通过[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)一次性将数据全部加载到StarRocks中。Spark连接器基于Spark DataSource V2实现。通过Spark DataFrames或Spark SQL可以创建一个DataSource。支持批处理和结构化流式处理模式。

> **注意**
>
> 只有对StarRocks表具有SELECT和INSERT权限的用户才能将数据加载到表中。您可以按照[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)中提供的说明为用户授予这些权限。

## 版本要求

| Spark连接器 | Spark       | StarRocks | Java | Scala |
| ----------- | ----------- | --------- | ---- | ----- |
| 1.1.1       | 3.2, 3.3或3.4 | 2.5及更高版本 | 8 | 2.12 |
| 1.1.0       | 3.2, 3.3或3.4 | 2.5及更高版本 | 8 | 2.12 |

> **注意**
>
> - 有关Spark连接器不同版本之间行为变化的详细信息，请参阅[升级Spark连接器](#upgrade-spark-connector)。
> - 从1.1.1版本开始，Spark连接器不再提供MySQL JDBC驱动，请手动将驱动程序导入spark类路径。您可以在[MySQL站点](https://dev.mysql.com/downloads/connector/j/)或[Maven中央仓库](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)上找到驱动程序。

## 获取Spark连接器

您可以通过以下方式获取Spark连接器JAR文件：

- 直接下载已编译的Spark连接器JAR文件。
- 将Spark连接器作为Maven项目的依赖项，然后下载JAR文件。
- 自行编译Spark连接器的源代码至JAR文件。

Spark连接器JAR文件的命名格式为`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`。

例如，如果您在环境中安装了Spark 3.2和Scala 2.12，并且希望使用Spark连接器1.1.0，则可以使用`starrocks-spark-connector-3.2_2.12-1.1.0.jar`。

> **注意**
>
> 一般来说，Spark连接器的最新版本仅与最近的三个Spark版本保持兼容。

### 下载已编译的JAR文件

可直接从[Maven中央仓库](https://repo1.maven.org/maven2/com/starrocks)下载对应版本的Spark连接器JAR。

### Maven依赖

1. 在Maven项目的`pom.xml`文件中，按照以下格式将Spark连接器作为依赖项添加到项目中。将`spark_version`、`scala_version`和`connector_version`替换为相应的版本。

    ```xml
    <dependency>
      <groupId>com.starrocks</groupId>
      <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
      <version>${connector_version}</version>
    </dependency>
    ```

2. 例如，如果您的环境中Spark的版本为3.2，Scala的版本为2.12，且选择了Spark连接器1.1.0，则需添加以下依赖：

    ```xml
    <dependency>
      <groupId>com.starrocks</groupId>
      <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
      <version>1.1.0</version>
    </dependency>
    ```

### 自行编译

1. 下载[Spark连接器包](https://github.com/StarRocks/starrocks-connector-for-apache-spark)。
2. 执行以下命令，将Spark连接器的源代码编译成JAR文件。注意，`spark_version`应替换为相应的Spark版本。

      ```bash
      sh build.sh <spark_version>
      ```

   例如，如果您的环境中Spark的版本为3.2，则需执行以下命令：

      ```bash
      sh build.sh 3.2
      ```

3. 进入`target/`目录，查找编译后的Spark连接器JAR文件，例如`starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar`。

> **注意**
>
> 未正式发布的Spark连接器名称都包含`SNAPSHOT`后缀。

## 参数

| 参数                                        | 必需   | 默认值        | 描述                                                         |
| ------------------------------------------- | ------ | ------------- | ------------------------------------------------------------ |
| starrocks.fe.http.url                        | 是     | 无           | StarRocks集群中FE的HTTP URL。可以指定多个URL，各URL之间需用逗号（,）分隔。格式：`<fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>`。从1.1.1版本开始，还可以在URL前加上`http://`前缀，如 `http://<fe_host1>:<fe_http_port1>,http://<fe_host2>:<fe_http_port2>`。|
| starrocks.fe.jdbc.url                        | 是     | 无           | 用于连接FE的MySQL服务器的地址。格式：`jdbc:mysql://<fe_host>:<fe_query_port>`。|
| starrocks.table.identifier                   | 是     | 无           | StarRocks表的名称。格式：`<database_name>.<table_name>`。|
| starrocks.user                               | 是     | 无           | 您StarRocks集群账户的用户名。用户需要在StarRocks表上拥有[SELECT和INSERT权限](../sql-reference/sql-statements/account-management/GRANT.md)。          |
| starrocks.password                           | 是     | 无           | 您StarRocks集群账户的密码。                                   |
| starrocks.write.label.prefix                 | 否     | spark-        | Stream Load使用的标签前缀。                                |
| starrocks.write.enable.transaction-stream-load | 否 | TRUE | 是否使用[Stream Load事务接口](../loading/Stream_Load_transaction_interface.md)加载数据。需要StarRocks v2.5或更高版本。此功能可以在事务中加载更多数据，占用更少的内存，并提高性能。<br/> **注意：** 自1.1.1版本起，该参数仅在`starrocks.write.max.retries`的值为非正数时生效，因为Stream Load事务接口不支持重试。 |
| starrocks.write.buffer.size                  | 否     | 104857600     | 可在一次发送到StarRocks之前在内存中累积的最大数据量。将此参数设置为较大的值可以提高加载性能，但可能会增加加载延迟。 |
| starrocks.write.buffer.rows                  | 否     | Integer.MAX_VALUE | 从1.1.1版本开始支持。在一次发送到StarRocks之前在内存中累积的最大行数。 |
| starrocks.write.flush.interval.ms            | 否     | 300000        | 发送数据到StarRocks的时间间隔。此参数用于控制加载延迟。                |
| starrocks.write.max.retries                  | 否     | 3             | 从1.1.1版本开始支持。如果加载失败，连接器重试执行同一批数据的Stream Load的次数。<br/> **注意：** 因为Stream Load事务接口不支持重试。如果此参数为正数，该连接器总是使用Stream Load接口，且会忽略`starrocks.write.enable.transaction-stream-load`的值。|
| starrocks.write.retry.interval.ms            | 否     | 10000         | 从1.1.1版本开始支持。如果加载失败，重试执行同一批数据的Stream Load的时间间隔。|
| starrocks.columns                            | 否     | 无           | 您要加载数据的StarRocks表列。您可以指定多个列，各列之间需用逗号（,）分隔，例如`"col0,col1,col2"`。|
| starrocks.column.types                         | 否       | 无             | 自 1.1.1 版本起支持。自定义Spark的列数据类型，而不是使用从StarRocks表和[默认映射](#数据类型映射-介绍)中推断出的默认值。参数值是一个以 DDL 格式表示的模式，与 Spark [StructType#toDDL](https://github.com/apache/spark/blob/master/sql/api/src/main/scala/org/apache/spark/sql/types/StructType.scala#L449) 的输出相同，比如 `col0 INT, col1 STRING, col2 BIGINT`。请注意，您只需要指定需要自定义的列。一个用例是将数据加载到[BITMAP](#将数据加载到位图类型的列)或[HLL](#将数据加载到HLL类型的列)类型的列中。|
| starrocks.write.properties.*                   | 否       | 无             | 用于控制流式加载行为的参数。例如，参数 `starrocks.write.properties.format` 指定要加载的数据的格式，如 CSV 或 JSON。支持的参数及其描述，请参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。 |
| starrocks.write.properties.format              | 否       | CSV           | Spark连接器在数据发送到StarRocks之前，将每个数据批次转换为的文件格式。有效值：CSV 和 JSON。|
| starrocks.write.properties.row_delimiter       | 否       | \n            | CSV格式数据的行分隔符。|
| starrocks.write.properties.column_separator    | 否       | \t            | CSV格式数据的列分隔符。|
| starrocks.write.num.partitions                 | 否       | 无             | Spark可以并行写入数据的分区数。当数据量较小时，可以减少分区数以降低加载的并发度和频率。此参数的默认值由Spark确定。然而，此方法可能会导致Spark Shuffle成本。|
| starrocks.write.partition.columns              | 否       | 无             | Spark中的分区列。只有在指定了 `starrocks.write.num.partitions` 的情况下，该参数才会生效。如果未指定该参数，则使用正在写入的所有列进行分区。|
| starrocks.timezone | 否 | JVM默认时区 | 自 1.1.1 版本起支持。用于将Spark的 `TimestampType` 转换为StarRocks的 `DATETIME` 所使用的时区。默认值是 `ZoneId#systemDefault()` 返回的JVM时区。格式可以是一个时区名，如 `Asia/Shanghai`，或一个区偏移，如 `+08:00`。 |

## Spark和StarRocks之间的数据类型映射

- 默认数据类型映射如下：

  | Spark数据类型 | StarRocks数据类型                                           |
  | -------------- | ------------------------------------------------------------ |
  | BooleanType    | BOOLEAN                                                      |
  | ByteType       | TINYINT                                                      |
  | ShortType      | SMALLINT                                                     |
  | IntegerType    | INT                                                          |
  | LongType       | BIGINT                                                       |
  | StringType     | LARGEINT                                                     |
  | FloatType      | FLOAT                                                        |
  | DoubleType     | DOUBLE                                                       |
  | DecimalType    | DECIMAL                                                      |
  | StringType     | CHAR                                                         |
  | StringType     | VARCHAR                                                      |
  | StringType     | STRING                                                       |
  | DateType       | DATE                                                         |
  | TimestampType  | DATETIME                                                     |
  | ArrayType      | ARRAY <br /> **注意：** <br /> **自 1.1.1 版本起支持。详细步骤，请参见[将数据加载到数组类型的列](#将数据加载到数组类型的列)。 |

- 您还可以自定义数据类型映射。

  例如，StarRocks表包含BITMAP和HLL列，但Spark不支持这两种数据类型。您需要自定义Spark中对应的数据类型。详细步骤，请参见将数据加载到[BITMAP](#将数据加载到位图类型的列)和[HLL](#将数据加载到HLL类型的列)列。**BITMAP 和 HLL 自 1.1.1 版本起支持**。

## 升级Spark连接器

### 从 1.1.0 版本升级到 1.1.1

- 自 1.1.1 版本起，Spark连接器不再提供 `mysql-connector-java`，这是MySQL的官方JDBC驱动，因为 `mysql-connector-java` 使用了GPL许可证的限制。
  然而，Spark连接器仍然需要MySQL JDBC驱动连接到StarRocks获取表的元数据，因此您需要手动将驱动程序添加到Spark类路径中。您可以在[MySQL官方网站](https://dev.mysql.com/downloads/connector/j/)或[Maven中央仓库](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)找到该驱动程序。
- 自 1.1.1 版本起，连接器默认使用流式加载接口，而不是 1.1.0 版本中的流式加载事务接口。如果仍然想使用流式加载事务接口，可以将选项 `starrocks.write.max.retries` 设置为 `0`。详细信息，请参见 `starrocks.write.enable.transaction-stream-load` 和 `starrocks.write.max.retries` 的描述。

## 示例


以下示例展示了如何使用Spark连接器将数据加载到StarRocks表中，使用Spark DataFrames或Spark SQL。Spark DataFrames同时支持批量处理和结构化流处理等模式。

更多示例，请参见[Spark连接器示例](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/main/src/test/java/com/starrocks/connector/spark/examples)。

### 准备工作

#### 创建一个StarRocks表

创建一个名为 `test` 的数据库，然后创建一个名为 `score_board` 的主键表。

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

#### 设置您的Spark环境

请注意，以下示例在Spark 3.2.4中运行，并使用 `spark-shell`、`pyspark` 和 `spark-sql`。在运行示例之前，请确保将Spark连接器JAR文件放在 `$SPARK_HOME/jars` 目录中。

### 使用Spark DataFrames加载数据

以下两个示例解释了如何使用Spark DataFrames的批处理或结构化流处理模式加载数据。

#### 批量处理

在内存中构造数据，并将数据加载到StarRocks表中。

1. 您可以使用Scala或Python编写Spark应用程序。

  对于Scala，您可以在 `spark-shell` 中运行以下代码片段：

   ```Scala
   // 1. 从序列创建DataFrame。
   val data = Seq((1, "starrocks", 100), (2, "spark", 100))
   val df = data.toDF("id", "name", "score")

   // 2. 配置格式为 "starrocks" 和以下选项，将数据写入StarRocks。
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

   对于Python，您可以在 `pyspark` 中运行以下代码片段：

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

    # 2. 配置格式为 "starrocks" 和以下选项，将数据写入StarRocks。
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

构建从CSV文件读取数据并将数据加载到StarRocks表中的流式读取。

1. 在目录`csv-data`中，创建一个名为`test.csv`的CSV文件，其中包含如下数据：

    ```csv
    3,starrocks,100
    4,spark,100
    ```

2. 您可以使用Scala或Python编写Spark应用程序。

   对于Scala，请在`spark-shell`中运行以下代码片段：

   ```Scala
   import org.apache.spark.sql.types.StructType

   // 1. 从CSV创建数据框架。
   val schema = (new StructType()
           .add("id", "integer")
           .add("name", "string")
           .add("score", "integer")
       )
   val df = (spark.readStream
           .option("sep", ",")
           .schema(schema)
           .format("csv") 
           // 用实际目录“csv-data”替换
           .load("/path/to/csv-data")
       )
   
   // 2. 通过将格式配置为“starrocks”和以下选项，将数据写入StarRocks。您需要根据自己的环境修改选项。
   val query = (df.writeStream.format("starrocks")
           .option("starrocks.fe.http.url", "127.0.0.1:8030")
           .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
           .option("starrocks.table.identifier", "test.score_board")
           .option("starrocks.user", "root")
           .option("starrocks.password", "")
           // 用实际检查点目录替换
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
   
   # 1. 从CSV创建数据框架。
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
       # 用实际目录“csv-data”替换
       .load("/path/to/csv-data")
   )

   # 2. 通过将格式配置为“starrocks”和以下选项，将数据写入StarRocks。您需要根据自己的环境修改选项。
   query = (
       df.writeStream.format("starrocks")
       .option("starrocks.fe.http.url", "127.0.0.1:8030")
       .option("starrocks.fe.jdbc.url", "jdbc:mysql://127.0.0.1:9030")
       .option("starrocks.table.identifier", "test.score_board")
       .option("starrocks.user", "root")
       .option("starrocks.password", "")
       # 用实际检查点目录替换
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

以下示例说明了如何使用Spark SQL通过使用[Spark SQL CLI](https://spark.apache.org/docs/latest/sql-distributed-sql-engine-spark-sql-cli.html)中的`INSERT INTO`语句加载数据。

1. 在`spark-sql`中执行以下SQL语句：

   ```SQL
   -- 1. 通过配置数据源为`starrocks`和以下选项，创建一个表。您需要根据自己的环境修改选项。
   CREATE TABLE `score_board`
   USING starrocks
   OPTIONS(
   "starrocks.fe.http.url"="127.0.0.1:8030",
   "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:9030",
   "starrocks.table.identifier"="test.score_board",
   "starrocks.user"="root",
   "starrocks.password"=""
   );

   -- 2. 往表中插入两行数据。
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

### 加载数据到主键表

本节将展示如何将数据加载到StarRocks主键表，以实现部分更新和条件更新。您可以参考[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)了解这些功能的详细介绍。这些示例使用Spark SQL。

#### 准备工作

在StarRocks中创建一个名为`test`的数据库，并在其中创建一个名为`score_board`的主键表。

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

本示例将展示如何仅通过加载更新 `name` 列中的数据：

1. 在MySQL客户端向StarRocks表中插入初始数据。

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

2. 在Spark SQL客户端创建名为 `score_board` 的Spark表。

   - 将选项`starrocks.write.properties.partial_update`设置为`true`，告知连接器进行部分更新。
   - 将选项`starrocks.columns`设置为`"id,name"`，告知连接器要写入哪些列。

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

3. 在Spark SQL客户端将数据插入表中，并仅更新`name`列。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'spark-update');
   ```

4. 在MySQL客户端查询StarRocks表。

   您会发现只有`name`的值发生了变化，`score`的值没有变化。

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

#### 条件更新

本示例将展示如何根据列 `score` 的值进行条件更新。只有当`score`的新值大于或等于旧值时，`id`的更新才会生效。
1. 在MySQL客户端向StarRocks表插入初始数据。

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

2. 以以下方式创建Spark表`score_board`。

   - 将选项`starrocks.write.properties.merge_condition`设置为`score`，告知连接器使用列`score`作为条件。
   - 确保Spark连接器使用Stream Load接口加载数据，而不是Stream Load事务接口，因为后者不支持此功能。

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

3. 在Spark SQL客户端向表中插入数据，并更新`id`为1的行的score值较小，`id`为2的行的score值较大。

   ```SQL
   INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'spark-update', 101);
   ```

4. 在MySQL客户端查询StarRocks表。

   您可以看到仅`id`为2的行发生了变化，`id`为1的行没有变化。

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

### 将数据加载到位图类型的列

[`BITMAP`](../sql-reference/sql-statements/data-types/BITMAP.md)经常用于加速精确计数去重，例如计算UV，请参见[使用位图进行精确Count Distinct](../using_starrocks/Using_bitmap.md)。
这里我们以计算UV为例，展示了如何将数据加载到`BITMAP`类型列中。**`BITMAP`自版本1.1.1开始支持**。

1. 创建一个StarRocks聚合表。

   在数据库`test`中创建一个名为`page_uv`的聚合表，其中列`visit_users`被定义为`BITMAP`类型，并配置了聚合函数`BITMAP_UNION`。

    ```SQL
    CREATE TABLE `test`.`page_uv` (
      `page_id` INT NOT NULL COMMENT '页面ID',
      `visit_date` datetime NOT NULL COMMENT '访问时间',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT '用户ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```

2. 创建一个Spark表。

   Spark表的模式是从StarRocks表中推断出的，并且Spark不支持`BITMAP`类型。因此，您需要在Spark中自定义相应的列数据类型，例如`BIGINT`，通过配置选项`"starrocks.column.types"="visit_users BIGINT"`。在使用Stream Load加载数据时，连接器会使用[`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)函数将`BIGINT`类型的数据转换为`BITMAP`类型。

    在`spark-sql`中执行以下DDL：

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

3. 将数据加载到StarRocks表中。

    在`spark-sql`中运行以下DML：

    ```SQL
    INSERT INTO `page_uv` VALUES
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 13),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23),
       (1, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 33),
       (1, CAST('2020-06-23 02:30:30' AS TIMESTAMP), 13),
       (2, CAST('2020-06-23 01:30:30' AS TIMESTAMP), 23);
    ```

4. 从StarRocks表计算页面UV。

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

> **注意:**
>
> 连接器使用[`to_bitmap`](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)函数将Spark中`TINYINT`、`SMALLINT`、`INTEGER`和`BIGINT`类型的数据转换为StarRocks中的`BITMAP`类型，并对其他Spark数据类型使用[`bitmap_hash`](../sql-reference/sql-functions/bitmap-functions/bitmap_hash.md)函数。

### 将数据加载到HLL类型的列

[`HLL`](../sql-reference/sql-statements/data-types/HLL.md)可用于近似计数去重，请参见[使用HLL进行近似Count Distinct](../using_starrocks/Using_HLL.md)。

这里我们以计算UV为例，展示了如何将数据加载到`HLL`类型的列中。**`HLL`自版本1.1.1开始支持**。

1. 创建一个StarRocks聚合表。

   在数据库`test`中创建一个名为`hll_uv`的聚合表，其中列`visit_users`被定义为`HLL`类型，并配置了聚合函数`HLL_UNION`。

    ```SQL

    CREATE TABLE `hll_uv` (
    `page_id` INT NOT NULL COMMENT '页面ID',
    `visit_date` datetime NOT NULL COMMENT '访问时间',
    `visit_users` HLL HLL_UNION NOT NULL COMMENT '用户ID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`);
    ```


2. 创建一个Spark表。

   Spark表的模式是从StarRocks表中推断出的，并且Spark不支持`HLL`类型。因此，您需要在Spark中自定义相应的列数据类型，例如`BIGINT`，通过配置选项`"starrocks.column.types"="visit_users BIGINT"`。在使用Stream Load加载数据时，连接器会使用[`hll_hash`](../sql-reference/sql-functions/aggregate-functions/hll_hash.md)函数将`BIGINT`类型的数据转换为`HLL`类型。

    在`spark-sql`中执行以下DDL：

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

3. 将数据加载到StarRocks表中。

    在`spark-sql`中运行以下DML：

    ```SQL
    INSERT INTO `hll_uv` VALUES
       (3, CAST('2023-07-24 12:00:00' AS TIMESTAMP), 78),
       (4, CAST('2023-07-24 13:20:10' AS TIMESTAMP), 2),
       (3, CAST('2023-07-24 12:30:00' AS TIMESTAMP), 674);
    ```

4. 从 StarRocks 表中计算页面的 UV 数。

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

下面的示例说明了如何将数据加载到 [`ARRAY`](../sql-reference/sql-statements/data-types/Array.md) 类型的列中。

1. 创建一个 StarRocks 表。

   在数据库 `test` 中，创建一个主键表 `array_tbl`，包括一个 `INT` 列和两个 `ARRAY` 列。

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

   由于某些版本的 StarRocks 没有提供 `ARRAY` 列的元数据，连接器无法推断此列的相应 Spark 数据类型。但是，您可以在选项 `starrocks.column.types` 中明确指定列的相应 Spark 数据类型。在这个示例中，您可以配置选项为 `a0 ARRAY<STRING>,a1 ARRAY<ARRAY<INT>>`。

   在 `spark-shell` 中运行以下代码：

   ```scala
    val data = Seq(
       |  (1, Seq("hello", "starrocks"), Seq(Seq(1, 2), Seq(3, 4))),
       |  (2, Seq("hello", "spark"), Seq(Seq(5, 6, 7), Seq(8, 9, 10)))
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
   2 rows in set (0.01 sec)
   ```