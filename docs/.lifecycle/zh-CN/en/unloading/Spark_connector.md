---
displayed_sidebar: "Chinese"
---

# 使用 Spark 连接器从 StarRocks 读取数据

StarRocks 提供一个名为 StarRocks Connector for Apache Spark™（简称 Spark 连接器）的自研连接器，帮助您通过 Spark 从 StarRocks 表中读取数据。您可以使用 Spark 对从 StarRocks 读取的数据进行复杂处理和机器学习。

Spark 连接器支持三种读取方法：Spark SQL、Spark DataFrame 和 Spark RDD。

您可以使用 Spark SQL 在 StarRocks 表上创建临时视图，然后直接通过该临时视图从 StarRocks 表中读取数据。

您还可以将 StarRocks 表映射到 Spark DataFrame 或 Spark RDD，然后从 Spark DataFrame 或 Spark RDD 读取数据。我们建议使用 Spark DataFrame。

> **注意**
>
> 只有对 StarRocks 表具有 SELECT 权限的用户才能从该表中读取数据。您可以按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中提供的说明向用户授予该权限。

## 使用注意事项

- 您可以在从 StarRocks 读取数据之前对其进行筛选，从而减少传输的数据量。
- 如果读取数据的开销很大，您可以采用适当的表设计和筛选条件，防止 Spark 一次性读取过量的数据。如此一来，您可以减轻磁盘和网络连接的 I/O 压力，确保例行查询可以正常运行。

## 版本要求

| Spark 连接器 | Spark               | StarRocks          | Java | Scala |
|------------- | ------------------- | ------------------ | ---- | ----- |
| 1.1.1        | 3.2, 3.3, 3.4       | 2.5 及更高版本     | 8    | 2.12  |
| 1.1.0        | 3.2, 3.3, 3.4       | 2.5 及更高版本     | 8    | 2.12  |
| 1.0.0        | 3.x                 | 1.18 及更高版本    | 8    | 2.12  |
| 1.0.0        | 2.x                 | 1.18 及更高版本    | 8    | 2.11  |

> **注意**
>
> - 有关不同连接器版本之间行为变更，请参见[升级 Spark 连接器](#upgrade-spark-connector)。
> - 从版本 1.1.1 起，连接器不再提供 MySQL JDBC 驱动程序，您需要手动将驱动程序导入 Spark 类路径。您可以在 [Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/) 上找到该驱动程序。
> - 在 1.0.0 版本中，Spark 连接器仅支持从 StarRocks 读取数据。从 1.1.0 版本开始，Spark 连接器既支持从 StarRocks 读取数据，也支持向其写入数据。
> - 1.0.0 版本与 1.1.0 版本在参数和数据类型映射方面存在差异。请参见[升级 Spark 连接器](#upgrade-spark-connector)。
> - 通常情况下，1.0.0 版本不会添加新功能。我们建议您尽快升级 Spark 连接器。

## 获取 Spark 连接器

使用以下方法之一获取适合您业务需求的 Spark 连接器 **.jar** 包：

- 下载已编译的包。
- 使用 Maven 添加 Spark 连接器所需的依赖项（仅支持 Spark 连接器 1.1.0 及更高版本）。
- 手动编译包。

### Spark 连接器 1.1.0 及更高版本

Spark 连接器 **.jar** 包的命名格式如下：

`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

例如，如果您想要使用 Spark 3.2 和 Scala 2.12 的 Spark 连接器 1.1.0，您可以选择 `starrocks-spark-connector-3.2_2.12-1.1.0.jar`。

> **注意**
>
> 在一般情况下，最新的 Spark 连接器版本可以与最近的三个 Spark 版本一起使用。

#### 下载已编译的包

您可以在 [Maven 中央仓库](https://repo1.maven.org/maven2/com/starrocks) 获取各种版本的 Spark 连接器 **.jar** 包。

#### 添加 Maven 依赖项

配置 Spark 连接器所需的依赖项如下：

> **注意**
>
> 您必须将 `spark_version`、`scala_version` 和 `connector_version` 替换为您使用的 Spark 版本、Scala 版本和 Spark 连接器版本。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
  <version>${connector_version}</version>
</dependency>
```

例如，如果您想要使用 Spark 3.2 和 Scala 2.12 的 Spark 连接器 1.1.0，则配置依赖项如下：

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
  <version>1.1.0</version>
</dependency>
```

#### 手动编译包

1. 下载 [Spark 连接器代码](https://github.com/StarRocks/starrocks-connector-for-apache-spark)。

2. 使用以下命令编译 Spark 连接器：

   > **注意**
   >
   > 您必须将 `spark_version` 替换为您使用的 Spark 版本。

   ```shell
   sh build.sh <spark_version>
   ```

   例如，如果您想要使用 Spark 3.2 的 Spark 连接器，对 Spark 连接器进行编译如下：

   ```shell
   sh build.sh 3.2
   ```

3. 进入 `target/` 路径，在其中会生成类似 `starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar` 的 Spark 连接器 **.jar** 包。

   > **注意**
   >
   > 如果您使用的是尚未正式发布的 Spark 连接器版本，则生成的 Spark 连接器 **.jar** 包的名称会包含 `SNAPSHOT` 作为后缀。

### Spark 连接器 1.0.0

#### 下载已编译的包

- [Spark 2.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark2_2.11-1.0.0.jar)
- [Spark 3.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark3_2.12-1.0.0.jar)

#### 手动编译包

1. 下载 [Spark 连接器代码](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/spark-1.0)。

   > **注意**
   >
   > 您必须切换到 `spark-1.0` 分支。

2. 采取以下行动之一编译 Spark 连接器：

   - 如果您使用的是 Spark 2.x，运行以下命令将 Spark 连接器编译为默认适配 Spark 2.3.4 的版本：

     ```Plain
     sh build.sh 2
     ```

   - 如果您使用的是 Spark 3.x，运行以下命令将 Spark 连接器编译为默认适配 Spark 3.1.2 的版本：

     ```Plain
     sh build.sh 3
     ```

3. 进入 `output/` 路径，在其中生成了 `starrocks-spark2_2.11-1.0.0.jar` 文件。然后，将文件复制到 Spark 的类路径中：

   - 如果您的 Spark 集群在 `Local` 模式下运行，请将文件放入 `jars/` 路径。
   - 如果您的 Spark 集群在 `Yarn` 模式下运行，请将文件放入预部署包中。

只有在将文件放置到指定位置后，您才能使用 Spark 连接器从 StarRocks 读取数据。

## 参数

本节描述了您使用 Spark 连接器从 StarRocks 读取数据时需要配置的参数。

### 通用参数

以下参数适用于三种读取方法：Spark SQL、Spark DataFrame 和 Spark RDD。

| 参数                               | 默认值           | 描述                                                         |
| ---------------------------------- | ---------------- | ------------------------------------------------------------ |
| starrocks.fenodes                   | 无               | StarRocks 集群中 FE 的 HTTP URL。格式为 `<fe_host>:<fe_http_port>`。您可以指定多个 URL，URL 之间需用逗号（,）分隔。 |
| starrocks.table.identifier          | 无               | StarRocks 表的名称。格式为 `<database_name>.<table_name>`。     |
| starrocks.request.retries           | 3                | Spark 可重试发送读取请求到 StarRocks 的最大次数。                 |
| starrocks.request.connect.timeout.ms| 30000            | 发送到 StarRocks 的读取请求超时的最大时长。                       |
| starrocks.request.read.timeout.ms    | 30000             | 发送到StarRocks的请求读取超时的最长时间。 |
| starrocks.request.query.timeout.s    | 3600              | 从StarRocks查询数据的最长时间。默认超时时间为1小时。`-1`表示未指定超时时间。 |
| starrocks.request.tablet.size        | Integer.MAX_VALUE | 每个Spark RDD分区中分组的StarRocks tablet的数量。该参数的较小值表示将生成较大数量的Spark RDD分区。较大数量的Spark RDD分区意味着在Spark上有更高的并行性，但对StarRocks的压力更大。 |
| starrocks.batch.size                 | 4096              | 一次从BE读取的最大行数。增加该参数的值可以减少Spark与StarRocks之间建立的连接数量，从而减轻由网络延迟引起的额外时间开销。 |
| starrocks.exec.mem.limit             | 2147483648        | 每个查询允许使用的最大内存量。单位：字节。默认内存限制为2 GB。 |
| starrocks.deserialize.arrow.async    | false             | 指定是否支持将Arrow内存格式异步转换为Spark连接器迭代所需的RowBatches。 |
| starrocks.deserialize.queue.size     | 64                | 内部队列的大小，该队列保存了异步转换Arrow内存格式为RowBatches的任务。该参数在`starrocks.deserialize.arrow.async`设置为`true`时有效。 |
| starrocks.filter.query               | 无                | 你想要在StarRocks上过滤数据的条件。你可以指定多个过滤条件，它们必须由`and`连接。在数据被Spark读取之前，StarRocks基于指定的过滤条件过滤StarRocks表中的数据。 |
| starrocks.timezone | JVM默认时区 | 自1.1.1起支持。用于将StarRocks的`DATETIME`转换为Spark`TimestampType`的时区。默认值为`ZoneId#systemDefault()`返回的JVM的时区。格式可以是时区名称，如`Asia/Shanghai`，或区域偏移，如`+08:00`。 |

### Spark SQL和Spark DataFrame的参数

以下参数仅适用于Spark SQL和Spark DataFrame的读取方法。

| 参数                              | 默认值         | 描述                             |
| --------------------------------- | ------------- | ------------------------------ |
| starrocks.fe.http.url               | 无          | FE的HTTP IP地址。从Spark连接器1.1.0开始支持。该参数等同于`starrocks.fenodes`。你只需要配置其中一个。在1.1.0及以后的Spark连接器中，我们建议你使用`starrocks.fe.http.url`，因为`starrocks.fenodes`可能被弃用。 |
| starrocks.fe.jdbc.url               | 无          | 用于连接FE的MySQL服务器的地址。格式：`jdbc:mysql://<fe_host>:<fe_query_port>`。<br />**注意**<br />在1.1.0及以后的Spark连接器中，该参数是必需的。   |
| user                                | 无          | 你的StarRocks集群账号的用户名。用户需要StarRocks表上的[SELECT权限](../sql-reference/sql-statements/account-management/GRANT.md)。   |
| starrocks.user                      | 无          | 你的StarRocks集群账号的用户名。从Spark连接器1.1.0开始支持。该参数等同于`user`。你只需要配置其中一个。在1.1.0及以后的Spark连接器中，我们建议你使用`starrocks.user`，因为`user`可能被弃用。   |
| password                            | 无          | 你的StarRocks集群账号的密码。    |
| starrocks.password                  | 无          | 你的StarRocks集群账号的密码。从Spark连接器1.1.0开始支持。该参数等同于`password`。你只需要配置其中一个。在1.1.0及以后的Spark连接器中，我们建议你使用`starrocks.password`，因为`password`可能被弃用。   |
| starrocks.filter.query.in.max.count | 100           | 在谓词下推期间IN表达式支持的最大值数量。如果IN表达式中指定的值数超过此限制，则在Spark上处理IN表达式中指定的过滤条件。   |

### Spark RDD的参数

以下参数仅适用于Spark RDD的读取方法。

| 参数                       | 默认值  | 描述                             |
| --------------------------- | ------ | ------------------------------- |
| starrocks.request.auth.user | 无     | 你的StarRocks集群账号的用户名。 |
| starrocks.request.auth.password | 无 | 你的StarRocks集群账号的密码。 |
| starrocks.read.field        | 无     | 你想要读取数据的StarRocks表列。你可以指定多个列，它们必须用逗号(,)分隔。 |

## StarRocks和Spark之间的数据类型映射

### Spark连接器1.1.0及以后版本

| StarRocks数据类型 | Spark数据类型           |
|-------------------- |-------------------------- |
| BOOLEAN             | DataTypes.BooleanType     |
| TINYINT             | DataTypes.ByteType        |
| SMALLINT            | DataTypes.ShortType       |
| INT                 | DataTypes.IntegerType     |
| BIGINT              | DataTypes.LongType        |
| LARGEINT            | DataTypes.StringType      |
| FLOAT               | DataTypes.FloatType       |
| DOUBLE              | DataTypes.DoubleType      |
| DECIMAL             | DecimalType               |
| CHAR                | DataTypes.StringType      |
| VARCHAR             | DataTypes.StringType      |
| STRING              | DataTypes.StringType      |
| DATE                | DataTypes.DateType        |
| DATETIME            | DataTypes.TimestampType   |
| ARRAY               | 不支持的数据类型           |
| HLL                 | 不支持的数据类型           |
| BITMAP              | 不支持的数据类型           |

### Spark连接器1.0.0

| StarRocks数据类型  | Spark数据类型       |
| -------------------- | ---------------------- |
| BOOLEAN        | DataTypes.BooleanType  |
| TINYINT         | DataTypes.ByteType     |
| SMALLINT        | DataTypes.ShortType    |
| INT                 | DataTypes.IntegerType  |
| BIGINT              | DataTypes.LongType     |
| LARGEINT          | DataTypes.StringType   |
| FLOAT               | DataTypes.FloatType    |
| DOUBLE             | DataTypes.DoubleType   |
| DECIMAL        | DecimalType            |
| CHAR              | DataTypes.StringType   |
| VARCHAR       | DataTypes.StringType   |
| DATE              | DataTypes.StringType   |
| DATETIME      | DataTypes.StringType   |
| ARRAY             | 不支持的数据类型        |
| HLL                 | 不支持的数据类型        |
| BITMAP         | 不支持的数据类型        |

当直接使用DATE和DATETIME数据类型时，StarRocks使用的底层存储引擎的处理逻辑无法覆盖预期的时间范围。因此，Spark连接器将来自StarRocks的DATE和DATETIME数据类型映射为Spark的STRING数据类型，并生成与从StarRocks读取的日期和时间数据匹配的可读字符串文本。

## 升级Spark连接器

###  从1.0.0版本升级到1.1.0版本

- 自1.1.1起，Spark连接器不提供`mysql-connector-java`，这是MySQL的官方JDBC驱动程序，因为`mysql-connector-java`使用的GPL许可证的限制。
  但是，Spark连接器仍需要`mysql-connector-java`来连接StarRocks获取表的元数据，因此你需要手动将该驱动程序添加到Spark类路径中。你可以在[MySQL网站](https://dev.mysql.com/downloads/connector/j/)或[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)找到该驱动程序。

- 在1.1.0版本中，Spark连接器使用JDBC来访问StarRocks以获取更详细的表信息。因此，你必须配置`starrocks.fe.jdbc.url`。


- 在1.1.0版本中，一些参数被重命名。新旧参数都暂时保留。对于每对等价参数，你只需要配置其中一个，但我们建议你使用新参数，因为旧参数可能被弃用。
  - `starrocks.fenodes`被重命名为`starrocks.fe.http.url`。
  - `user`被重命名为`starrocks.user`。
  - `password`被重命名为`starrocks.password`。

- 在1.1.0版本中，基于Spark 3.x，某些数据类型的映射进行了调整：
  - StarRocks中的`DATE`映射为Spark中的`DataTypes.DateType`（原始为`DataTypes.StringType`）。
  - StarRocks中的`DATETIME`映射为Spark中的`DataTypes.TimestampType`（原始为`DataTypes.StringType`）。

## 示例

以下示例假定你在StarRocks集群中创建了名为`test`的数据库，并拥有`root`用户的权限。示例中的参数设置基于Spark连接器1.1.0。

### 数据示例

按照以下步骤准备一个示例表格：

1. 进入`test`数据库并创建名为`score_board`的表格。

   ```SQL
   MySQL [test]> CREATE TABLE `score_board`
   (
       `id` int(11) NOT NULL COMMENT "",
       `name` varchar(65533) NULL DEFAULT "" COMMENT "",
       `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`id`)
   PROPERTIES (
       "replication_num" = "3"
   );
   ```

2. 向`score_board`表格插入数据。

   ```SQL
   MySQL [test]> INSERT INTO score_board
   VALUES
       (1, 'Bob', 21),
       (2, 'Stan', 21),
       (3, 'Sam', 22),
       (4, 'Tony', 22),
       (5, 'Alice', 22),
       (6, 'Lucy', 23),
       (7, 'Polly', 23),
       (8, 'Tom', 23),
       (9, 'Rose', 24),
       (10, 'Jerry', 24),
       (11, 'Jason', 24),
       (12, 'Lily', 25),
       (13, 'Stephen', 25),
       (14, 'David', 25),
       (15, 'Eddie', 26),
       (16, 'Kate', 27),
       (17, 'Cathy', 27),
       (18, 'Judy', 27),
       (19, 'Julia', 28),
       (20, 'Robert', 28),
       (21, 'Jack', 29);
   ```

3. 查询`score_board`表格。

   ```SQL
   MySQL [test]> SELECT * FROM score_board;
   +------+---------+-------+
   | id   | name    | score |
   +------+---------+-------+
   |    1 | Bob     |    21 |
   |    2 | Stan    |    21 |
   |    3 | Sam     |    22 |
   |    4 | Tony    |    22 |
   |    5 | Alice   |    22 |
   |    6 | Lucy    |    23 |
   |    7 | Polly   |    23 |
   |    8 | Tom     |    23 |
   |    9 | Rose    |    24 |
   |   10 | Jerry   |    24 |
   |   11 | Jason   |    24 |
   |   12 | Lily    |    25 |
   |   13 | Stephen |    25 |
   |   14 | David   |    25 |
   |   15 | Eddie   |    26 |
   |   16 | Kate    |    27 |
   |   17 | Cathy   |    27 |
   |   18 | Judy    |    27 |
   |   19 | Julia   |    28 |
   |   20 | Robert  |    28 |
   |   21 | Jack    |    29 |
   +------+---------+-------+
   21 rows in set (0.01 sec)
   ```

### 使用Spark SQL读取数据

1. 在Spark目录中运行以下命令启动Spark SQL：

   ```Plain
   sh spark-sql
   ```

2. 运行以下命令，在`test`数据库的`score_board`表格上创建名为`spark_starrocks`的临时视图：

   ```SQL
   spark-sql> CREATE TEMPORARY VIEW spark_starrocks
              USING starrocks
              OPTIONS
              (
                  "starrocks.table.identifier" = "test.score_board",
                  "starrocks.fe.http.url" = "<fe_host>:<fe_http_port>",
                  "starrocks.fe.jdbc.url" = "jdbc:mysql://<fe_host>:<fe_query_port>",
                  "starrocks.user" = "root",
                  "starrocks.password" = ""
              );
   ```

3. 运行以下命令从临时视图读取数据：

   ```SQL
   spark-sql> SELECT * FROM spark_starrocks;
   ```

   Spark返回以下数据：

   ```SQL
   1        Bob        21
   2        Stan        21
   3        Sam        22
   4        Tony        22
   5        Alice        22
   6        Lucy        23
   7        Polly        23
   8        Tom        23
   9        Rose        24
   10        Jerry        24
   11        Jason        24
   12        Lily        25
   13        Stephen        25
   14        David        25
   15        Eddie        26
   16        Kate        27
   17        Cathy        27
   18        Judy        27
   19        Julia        28
   20        Robert        28
   21        Jack        29
   Time taken: 1.883 seconds, Fetched 21 row(s)
   22/08/09 15:29:36 INFO thriftserver.SparkSQLCLIDriver: Time taken: 1.883 seconds, Fetched 21 row(s)
   ```

### 使用Spark DataFrame读取数据

1. 在Spark目录中运行以下命令启动Spark Shell：

   ```Plain
   sh spark-shell
   ```

2. 运行以下命令在`test`数据库的`score_board`表格上创建名为`starrocksSparkDF`的DataFrame：

   ```Scala
   scala> val starrocksSparkDF = spark.read.format("starrocks")
              .option("starrocks.table.identifier", s"test.score_board")
              .option("starrocks.fe.http.url", s"<fe_host>:<fe_http_port>")
              .option("starrocks.fe.jdbc.url", s"jdbc:mysql://<fe_host>:<fe_query_port>")
              .option("starrocks.user", s"root")
              .option("starrocks.password", s"")
              .load()
   ```

3. 从DataFrame中读取数据。例如，如果要读取前10行，运行以下命令：

   ```Scala
   scala> starrocksSparkDF.show(10)
   ```

   Spark返回以下数据：

   ```Scala
   +---+-----+-----+
   | id| name|score|
   +---+-----+-----+
   |  1|  Bob|   21|
   |  2| Stan|   21|
   |  3|  Sam|   22|
   |  4| Tony|   22|
   |  5|Alice|   22|
   |  6| Lucy|   23|
   |  7|Polly|   23|
   |  8|  Tom|   23|
   |  9| Rose|   24|
   | 10|Jerry|   24|
   +---+-----+-----+
   only showing top 10 rows
   ```

   > **注意**
   >
   > 默认情况下，如果未指定要读取的行数，Spark返回前20行。

### 使用Spark RDD读取数据

1. 在Spark目录中运行以下命令启动Spark Shell：

   ```Plain
   sh spark-shell
   ```

2. 运行以下命令在`test`数据库的`score_board`表格上创建名为`starrocksSparkRDD`的RDD。

   ```Scala
   scala> import com.starrocks.connector.spark._
   scala> val starrocksSparkRDD = sc.starrocksRDD
              (
              tableIdentifier = Some("test.score_board"),
              cfg = Some(Map(
                  "starrocks.fenodes" -> "<fe_host>:<fe_http_port>",
                  "starrocks.request.auth.user" -> "root",
                  "starrocks.request.auth.password" -> ""
              ))
              )
   ```

3. 从RDD中读取数据。例如，如果要读取前10个元素，运行以下命令：

   ```Scala
   scala> starrocksSparkRDD.take(10)
   ```

   Spark返回以下数据：

   ```Scala
   res0: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24])
   ```

   要读取整个RDD，运行以下命令：

   ```Scala
   scala> starrocksSparkRDD.collect()
   ```

   Spark返回以下数据：
```scala
   |                                                |
   |   1:EXCHANGE                                   |
   |                                                |
   | PLAN FRAGMENT 1                                |
   |  OUTPUT EXPRS:                                 |
   |   PARTITION: RANDOM                            |
   |                                                |
   |   STREAM DATA SINK                             |
   |     EXCHANGE ID: 01                            |
   |     UNPARTITIONED                              |
   |                                                |
   |   0:OlapScanNode                               |
   |      TABLE: mytable                            |
   |      PREAGGREGATION: ON                        |
   |      partitions=1/3                            |
   |      rollup: mytable                           |
   |      tabletRatio=3/9                          |
   |      tabletList=41299                           |
   |      cardinality=1                             |
   |      avgRowSize=4.0                            |
   |      numNodes=0                                |
   +------------------------------------------------+
   23 rows in set (0.00 sec)
   ```
```python
   |                                                |
   |   1:交换                                   |
   |                                                |
   | 计划片段 1                                |
   |  输出表达式:                                 |
   |   分区: 随机                            |
   |                                                |
   |   流数据接收器                             |
   |     交换标识: 01                            |
   |     无分区                              |
   |                                                |
   |   0:Olap扫描节点                               |
   |      表: mytable                            |
   |      预聚合: 开启                        |
   |      谓词: 3: dt = '2022-01-02 08:00:00' |
   |      分区数=1/3                            |
   |      滚动: mytable                           |
   |      tablet比例=3/3                           |
   |      tablet列表=41297,41299,41301              |
   |      基数=1                             |
   |      平均行大小=20.0                           |
   |      节点数=0                                |
   +------------------------------------------------+
   27 行在集合中 (0.01 秒)
   ```

在这个示例中，只执行了分区修剪，而没有执行桶修剪。因此，Spark扫描了三个分区中的一个（如`分区数=1/3`所示），以及该分区中的所有Tablet（如`tablet比例=3/3`所示）。

### 桶修剪

1. 运行以下命令，在Spark目录中使用`starrocks.filter.query`参数指定一个用于桶修剪的过滤条件`k=1`，在`mytable`表格所属的`test`数据库上创建一个名为`df`的DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

2. 查看您的StarRocks集群的FE日志文件 **fe.log**，找到执行读取数据的SQL语句。示例：

   ```SQL
   2022-08-09 19:04:44,479 INFO (nioEventLoopGroup-3-16|208) [TableQueryPlanAction.executeWithoutPassword():126] 接收到源自外部服务的SQL语句 [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1]，用户为 [ user ['root'@'%']]，数据库为 [test] 表格为 [mytable]
   ```

3. 在`test`数据库中，使用EXPLAIN获取执行`select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1`语句的执行计划：

   ```Scala
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1;
   +------------------------------------------+
   | 执行计划                                |
   +------------------------------------------+
   | 计划片段 0                             |
   |  输出表达式:1: k | 2: b | 3: dt | 4: v |
   |   分区: 无分区                          |
   |                                          |
   |   结果接收器                             |
   |                                          |
   |   1:交换                               |
   |                                          |
   | 计划片段 1                            |
   |  输出表达式:                           |
   |   分区: 随机                        |
   |                                          |
   |   流数据接收器                      |
   |     交换标识: 01                      |
   |     无分区                        |
   |                                          |
   |   0:Olap扫描节点                  |
   |      表: mytable                    |
   |      预聚合: 开启                |
   |      谓词: 1: k = 1               |
   |      分区数=3/3                      |
   |      滚动: mytable                 |
   |      tablet比例=3/9                 |
   |      tablet列表=41299,41305,41311        |
   |      基数=1                           |
   |      平均行大小=20.0                     |
   |      节点数=0                          |
   +------------------------------------------+
   27 行在集合中 (0.01 秒)
   ```

在这个示例中，只执行了桶修剪，而没有执行分区修剪。因此，Spark扫描了三个包含数据的分区中的所有分区（如`分区数=3/3`所示），并扫描了所有表格（如`tablet比例=3/9`所示）以检索满足这些三个分区中的`k=1`筛选条件的哈希值。

### 分区修剪和桶修剪

1. 运行以下命令，在Spark目录中使用`starrocks.filter.query`参数指定两个用于桶修剪和分区修剪的过滤条件 `k=7` 和 `dt='2022-01-02 08:00:00'`，在`test`数据库上的`mytable`表格创建一个名为 `df` 的DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"")
          .option("password", s"")
          .option("starrocks.filter.query", "k=7 and dt='2022-01-02 08:00:00'")
          .load()
   ```

2. 查看您的StarRocks集群的FE日志文件 **fe.log**，找到执行读取数据的SQL语句。示例：

   ```SQL
   2022-08-09 19:06:34,939 INFO (nioEventLoopGroup-3-18|212) [TableQueryPlanAction.executeWithoutPassword():126] 接收到源自外部服务的SQL语句 [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00']，用户为 [ user ['root'@'%']]，数据库为 [test] 表格为 [mytable]
   ```

3. 在`test`数据库中，使用EXPLAIN获取执行`select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'`语句的执行计划：

   ```Scala
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00';
   +----------------------------------------------------------+
   | 执行计划                                                  |
   +----------------------------------------------------------+
   | 计划片段 0                                                |
   |  输出表达式:1: k | 2: b | 3: dt | 4: v                       |
   |   分区: 随机                                             |
   |                                                            |
   |   结果接收器                                              |
   |                                                            |
   |   0:Olap扫描节点                                         |
   |      表: mytable                                          |
   |      预聚合: 开启                                      |
   |      谓词: 1: k = 7, 3: dt = '2022-01-02 08:00:00' |
   |      分区数=1/3                                          |
   |      滚动: mytable                                       |
   |      tablet比例=1/3                                       |
   |      tablet列表=41301                                    |
   |      基数=1                                               |
   |      平均行大小=20.0                                     |
   |      节点数=0                                            |
   +----------------------------------------------------------+
   17 行在集合中 (0.00 秒)
   ```

在这个示例中，执行了分区修剪和桶修剪。因此，Spark只扫描了一个（如`分区数=1/3`所示）包含数据的三个分区中的分区，并且只扫描了一个（如`tablet比例=1/3`所示）该分区中的表格。

### 前缀索引过滤

1. 将更多数据记录插入到属于`test`数据库的`mytable`表格的一个分区中：

   ```Scala
   MySQL [test]> INSERT INTO mytable
   VALUES
       (1, 11, "2022-01-02 08:00:00", 111), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333);
   ```

2. 查询`mytable`表格：

   ```Scala
   MySQL [test]> SELECT * FROM mytable;
   +------+------+---------------------+------+
   | k    | b    | dt                  | v    |
   +------+------+---------------------+------+
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
```
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    2 |   22 | 2022-02-02 08:00:00 |  222 |
   |    3 |   33 | 2022-03-02 08:00:00 |  333 |
   +------+------+---------------------+------+
   共7行(0.01秒)

3. 运行以下命令，其中使用`starrocks.filter.query`参数指定前缀索引过滤的筛选条件`k=1`，在Spark目录中为属于`test`数据库的`mytable`表创建名为`df`的DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

4. 在`test`数据库中，将`is_report_success`设置为`true`以启用性能报告：

   ```SQL
   MySQL [test]> SET is_report_success = true;
   查询成功，受影响行数：0，用时0.00秒
   ```

5. 使用浏览器打开`http://<fe_host>:<http_http_port>/query`页面，查看带有`k=1`条件的`SELECT * FROM mytable`语句的性能报告。例如：

   ```SQL
   OLAP_SCAN (plan_node_id=0):
     CommonMetrics:
        - CloseTime: 1.255ms
        - OperatorTotalTime: 1.404ms
        - PeakMemoryUsage: 0.00 
        - PullChunkNum: 8
        - PullRowNum: 2
          - __MAX_OF_PullRowNum: 2
          - __MIN_OF_PullRowNum: 0
        - PullTotalTime: 148.60us
        - PushChunkNum: 0
        - PushRowNum: 0
        - PushTotalTime: 0ns
        - SetFinishedTime: 136ns
        - SetFinishingTime: 129ns
     UniqueMetrics:
        - Predicates: 1: k = 1
        - Rollup: mytable
        - Table: mytable
        - BytesRead: 88.00 B
          - __MAX_OF_BytesRead: 88.00 B
          - __MIN_OF_BytesRead: 0.00 
        - CachedPagesNum: 0
        - CompressedBytesRead: 844.00 B
          - __MAX_OF_CompressedBytesRead: 844.00 B
          - __MIN_OF_CompressedBytesRead: 0.00 
        - CreateSegmentIter: 18.582us
        - IOTime: 4.425us
        - LateMaterialize: 17.385us
        - PushdownPredicates: 3
        - RawRowsRead: 2
          - __MAX_OF_RawRowsRead: 2
          - __MIN_OF_RawRowsRead: 0
        - ReadPagesNum: 12
          - __MAX_OF_ReadPagesNum: 12
          - __MIN_OF_ReadPagesNum: 0
        - RowsRead: 2
          - __MAX_OF_RowsRead: 2
          - __MIN_OF_RowsRead: 0
        - ScanTime: 154.367us
        - SegmentInit: 95.903us
          - BitmapIndexFilter: 0ns
          - BitmapIndexFilterRows: 0
          - BloomFilterFilterRows: 0
          - ShortKeyFilterRows: 3
            - __MAX_OF_ShortKeyFilterRows: 3
            - __MIN_OF_ShortKeyFilterRows: 0
          - ZoneMapIndexFilterRows: 0
        - SegmentRead: 2.559us
          - BlockFetch: 2.187us
          - BlockFetchCount: 2
            - __MAX_OF_BlockFetchCount: 2
            - __MIN_OF_BlockFetchCount: 0
          - BlockSeek: 7.789us
          - BlockSeekCount: 2
            - __MAX_OF_BlockSeekCount: 2
            - __MIN_OF_BlockSeekCount: 0
          - ChunkCopy: 25ns
          - DecompressT: 0ns
          - DelVecFilterRows: 0
          - IndexLoad: 0ns
          - PredFilter: 353ns
          - PredFilterRows: 0
          - RowsetsReadCount: 7
          - SegmentsReadCount: 3
            - __MAX_OF_SegmentsReadCount: 2
            - __MIN_OF_SegmentsReadCount: 0
          - TotalColumnsDataPageCount: 8
            - __MAX_OF_TotalColumnsDataPageCount: 8
            - __MIN_OF_TotalColumnsDataPageCount: 0
        - UncompressedBytesRead: 508.00 B
          - __MAX_OF_UncompressedBytesRead: 508.00 B
          - __MIN_OF_UncompressedBytesRead: 0.00 
   ```

```