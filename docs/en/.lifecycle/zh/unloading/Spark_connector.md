---
displayed_sidebar: English
---

# 使用 Spark 连接器从 StarRocks 读取数据

StarRocks 提供了一款名为 StarRocks Connector for Apache Spark™（简称 Spark 连接器）的自研连接器，帮助您使用 Spark 从 StarRocks 表中读取数据。您可以利用 Spark 对从 StarRocks 读取的数据进行复杂处理和机器学习。

Spark 连接器支持三种读取方法：Spark SQL、Spark DataFrame 和 Spark RDD。

您可以使用 Spark SQL 在 StarRocks 表上创建临时视图，然后直接使用该临时视图从 StarRocks 表中读取数据。

您也可以将 StarRocks 表映射到 Spark DataFrame 或 Spark RDD，然后从 Spark DataFrame 或 Spark RDD 中读取数据。我们建议使用 Spark DataFrame。

> **注意**
>
> 只有对 StarRocks 表拥有 SELECT 权限的用户才能读取该表的数据。您可以按照[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)中提供的说明向用户授予权限。

## 使用说明

- 您可以在读取数据之前对 StarRocks 上的数据进行过滤，从而减少传输的数据量。
- 如果读取数据的开销很大，可以采用适当的表设计和过滤条件来防止 Spark 一次读取过多的数据。因此，您可以减少磁盘和网络连接上的 I/O 压力，从而确保例行查询可以正常运行。

## 版本要求

| Spark 连接器 | Spark         | StarRocks       | Java | Scala |
|---------------- | ------------- | --------------- | ---- | ----- |
| 1.1.1           | 3.2, 3.3, 3.4 | 2.5 及更高版本   | 8    | 2.12  |
| 1.1.0           | 3.2, 3.3, 3.4 | 2.5 及更高版本   | 8    | 2.12  |
| 1.0.0           | 3.x           | 1.18 及更高版本  | 8    | 2.12  |
| 1.0.0           | 2.x           | 1.18 及更高版本  | 8    | 2.11  |

> **注意**
>
> - 有关不同连接器版本之间的行为更改，请参阅[升级 Spark 连接器](#upgrade-spark-connector)。
> - 从版本 1.1.1 开始，连接器不再提供 MySQL JDBC 驱动程序，您需要手动将驱动程序导入 Spark 类路径。您可以在[Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)上找到该驱动程序。
> - 在 1.0.0 版本中，Spark 连接器仅支持从 StarRocks 读取数据。从 1.1.0 版本开始，Spark 连接器支持从 StarRocks 读取数据和向 StarRocks 写入数据。
> - 版本 1.0.0 与版本 1.1.0 在参数和数据类型映射方面存在差异。请参阅[升级 Spark 连接器](#upgrade-spark-connector)。
> - 通常情况下，版本 1.0.0 不会添加任何新功能。我们建议您尽快升级 Spark 连接器。

## 获取 Spark 连接器

通过以下两种方式获取适合业务需求的 Spark 连接器 **.jar** 包：

- 下载已编译的软件包。
- 使用 Maven 添加 Spark 连接器所需的依赖项。（此方法仅支持 Spark 连接器 1.1.0 及更高版本。）
- 手动编译包。

### Spark 连接器 1.1.0 及更高版本

Spark 连接器 **.jar** 包的命名格式如下：

`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

例如，如果要使用 Spark 3.2 和 Scala 2.12 与 Spark 连接器 1.1.0，您可以选择 `starrocks-spark-connector-3.2_2.12-1.1.0.jar`。

> **注意**
>
> 通常情况下，最新的 Spark 连接器版本可以与最新的三个 Spark 版本一起使用。

#### 下载已编译的软件包

您可以在[Maven 中央存储库](https://repo1.maven.org/maven2/com/starrocks)获取各种版本的 Spark 连接器 **.jar** 包。

#### 添加 Maven 依赖项

配置 Spark 连接器所需的依赖项，如下所示：

> **注意**
>
> 您必须将 `spark_version`、 `scala_version` 和 `connector_version` 替换为您使用的 Spark 版本、Scala 版本和 Spark 连接器版本。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
  <version>${connector_version}</version>
</dependency>
```

例如，如果要使用 Spark 3.2 和 Scala 2.12 与 Spark 连接器 1.1.0，请配置依赖项如下：

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
  <version>1.1.0</version>
</dependency>
```

#### 手动编译包

1. 下载[Spark 连接器代码](https://github.com/StarRocks/starrocks-connector-for-apache-spark)。

2. 使用以下命令编译 Spark 连接器：

   > **注意**
   >
   > 您必须将 `spark_version` 替换为您使用的 Spark 版本。

   ```shell
   sh build.sh <spark_version>
   ```

   例如，如果要使用 Spark 3.2，编译 Spark 连接器的命令如下：

   ```shell
   sh build.sh 3.2
   ```

3. 进入 `target/` 路径，编译时将生成类似 `starrocks-spark-connector-3.2_2.12-1.1.0-SNAPSHOT.jar` 的 Spark 连接器 **.jar** 包。

   > **注意**
   >
   > 如果您使用的是未正式发布的 Spark 连接器版本，生成的 Spark 连接器 **.jar** 包的名称将包含 `SNAPSHOT` 作为后缀。

### Spark 连接器 1.0.0

#### 下载已编译的软件包

- [Spark 2.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark2_2.11-1.0.0.jar)
- [Spark 3.x](https://cdn-thirdparty.starrocks.com/spark/starrocks-spark3_2.12-1.0.0.jar)

#### 手动编译包

1. 下载[Spark 连接器代码](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/spark-1.0)。

   > **注意**
   >
   > 您必须切换到 `spark-1.0` 分支。

2. 执行以下操作来编译 Spark 连接器：

   - 如果您使用的是 Spark 2.x，请运行以下命令，默认情况下，此命令将编译适用于 Spark 2.3.4 的 Spark 连接器：

     ```Plain
     sh build.sh 2
     ```

   - 如果您使用的是 Spark 3.x，请运行以下命令，默认情况下，此命令将编译适用于 Spark 3.1.2 的 Spark 连接器：

     ```Plain
     sh build.sh 3
     ```

3. 进入 `output/` 路径，在此路径下会生成 `starrocks-spark2_2.11-1.0.0.jar` 文件。然后，将该文件复制到 Spark 的类路径中：

   - 如果您的 Spark 集群以 `Local` 模式运行，请将文件放入 `jars/` 路径中。
   - 如果您的 Spark 集群以 `Yarn` 模式运行，请将文件放入预部署包中。

只有将文件放置到指定位置后，才能使用 Spark 连接器从 StarRocks 读取数据。

## 参数

本节描述了使用 Spark 连接器从 StarRocks 读取数据时需要配置的参数。

### 通用参数

以下参数适用于所有三种读取方法：Spark SQL、Spark DataFrame 和 Spark RDD。

| 参数                            | 默认值     | 描述                                                    |
| ------------------------------------ | ----------------- | ------------------------------------------------------------ |
| starrocks.fenodes                    | 无              | StarRocks 集群中 FE 的 HTTP URL。格式为 `<fe_host>:<fe_http_port>`。您可以指定多个 URL，用逗号（,）分隔。 |
| starrocks.table.identifier           | 无              | StarRocks 表的名称。格式为 `<database_name>.<table_name>`。 |
| starrocks.request.retries            | 3                 | Spark 可以重试向 StarRocks 发送读取请求的最大次数。 |
| starrocks.request.connect.timeout.ms | 30000             | 发送到 StarRocks 的读取请求超时的最长时间。 |
| starrocks.request.read.timeout.ms    | 30000             | 发送到 StarRocks 的请求的读取超时的最长时间。 |
| starrocks.request.query.timeout.s    | 3600              | 从 StarRocks 查询数据的最长超时时间。默认超时时间为 1 小时。`-1` 表示未指定超时时间。 |
| starrocks.request.tablet.size        | Integer.MAX_VALUE | 每个 Spark RDD 分区中分组的 StarRocks Tablet 数量。此参数的值越小，表示将生成较多的 Spark RDD 分区。Spark RDD 分区数量越多，Spark 的并行度就越高，但 StarRocks 的压力也越大。 |
| starrocks.batch.size                 | 4096              | 一次可以从 BE 读取的最大行数。增加该参数的值可以减少 Spark 和 StarRocks 之间建立的连接数，从而减轻网络延迟带来的额外时间开销。 |
| starrocks.exec.mem.limit             | 2147483648        | 每个查询允许的最大内存量。单位：字节。默认内存限制为 2 GB。 |
| starrocks.deserialize.arrow.async    | false             | 指定是否支持将 Arrow 内存格式异步转换为 Spark 连接器迭代所需的 RowBatches。 |
| starrocks.deserialize.queue.size     | 64                | 内部队列的大小，用于保存用于将 Arrow 内存格式异步转换为 RowBatches 的任务。此参数在设置为 `true` 时有效 `starrocks.deserialize.arrow.async`。 |
| starrocks.filter.query               | 无              | 筛选 StarRocks 数据所依据的条件。您可以指定多个筛选条件，这些筛选条件必须用 `and` 连接。StarRocks 会根据指定的过滤条件过滤 StarRocks 表中的数据，然后再被 Spark 读取。 |
| starrocks.timezone | JVM 的默认时区 | 从 1.1.1 版本开始支持。用于将 StarRocks 的 `DATETIME` 转换为 Spark 的 `TimestampType`。默认值是 `ZoneId#systemDefault()` 返回的 JVM 时区。格式可以是时区名称（如 `Asia/Shanghai`），也可以是时区偏移量（如 `+08:00`）。 |

### Spark SQL 和 Spark DataFrame 的参数

以下参数仅适用于 Spark SQL 和 Spark DataFrame 读取方法。

| 参数                           | 默认值 | 描述                                                    |
| ----------------------------------- | ------------- | ------------------------------------------------------------ |

| starrocks.fe.http.url               | 无            | FE 的 HTTP IP 地址。从 Spark 连接器 1.1.0 开始支持此参数。此参数等效于 `starrocks.fenodes`。您只需要配置其中之一。在 Spark 连接器 1.1.0 及更高版本中，建议使用 `starrocks.fe.http.url`，因为 `starrocks.fenodes` 可能已被弃用。 |
| starrocks.fe.jdbc.url               | 无            | 用于连接到 FE 的 MySQL 服务器的地址。格式： `jdbc:mysql://<fe_host>:<fe_query_port>`。<br />**注意**<br />在 Spark 连接器 1.1.0 及更高版本中，此参数为必填项。   |
| 用户                                | 无            | 您 StarRocks 集群账户的用户名。用户需要在 StarRocks 表上拥有 [SELECT 权限](../sql-reference/sql-statements/account-management/GRANT.md)。  |
| starrocks.user                      | 无            | 您 StarRocks 集群账户的用户名。从 Spark 连接器 1.1.0 开始支持此参数。此参数等效于 `user`。您只需要配置其中之一。在 Spark 连接器 1.1.0 及更高版本中，建议使用 `starrocks.user`，因为 `user` 可能已被弃用。   |
| 密码                                | 无            | 您 StarRocks 集群账户的密码。    |
| starrocks.password                  | 无            | 您 StarRocks 集群账户的密码。从 Spark 连接器 1.1.0 开始支持此参数。此参数等效于 `password`。您只需要配置其中之一。在 Spark 连接器 1.1.0 及更高版本中，建议使用 `starrocks.password`，因为 `password` 可能已被弃用。   |
| starrocks.filter.query.in.max.count | 100           | 谓词下推期间 IN 表达式支持的最大值数。如果 IN 表达式中指定的值个数超过此限制，则在 Spark 上处理 IN 表达式中指定的筛选条件。   |

### Spark RDD 的参数

以下参数仅适用于 Spark RDD 读取方法。

| 参数                       | 默认值 | 描述                                                    |
| ------------------------------- | ------------- | ------------------------------------------------------------ |
| starrocks.request.auth.user     | 无            | 您 StarRocks 集群账户的用户名。              |
| starrocks.request.auth.password | 无            | 您 StarRocks 集群账户的密码。              |
| starrocks.read.field            | 无            | 您要读取数据的 StarRocks 表列。您可以指定多个列，它们必须用逗号（,）分隔。 |

## StarRocks 和 Spark 之间的数据类型映射

### Spark 连接器 1.1.0 及更高版本

| StarRocks 数据类型 | Spark 数据类型           |
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
| ARRAY               | 不支持的数据类型      |
| HLL                 | 不支持的数据类型      |
| BITMAP              | 不支持的数据类型      |

### Spark 连接器 1.0.0

| StarRocks 数据类型  | Spark 数据类型        |
| -------------------- | ---------------------- |
| BOOLEAN              | DataTypes.BooleanType  |
| TINYINT              | DataTypes.ByteType     |
| SMALLINT             | DataTypes.ShortType    |
| INT                  | DataTypes.IntegerType  |
| BIGINT               | DataTypes.LongType     |
| LARGEINT             | DataTypes.StringType   |
| FLOAT                | DataTypes.FloatType    |
| DOUBLE               | DataTypes.DoubleType   |
| DECIMAL              | DecimalType            |
| CHAR                 | DataTypes.StringType   |
| VARCHAR              | DataTypes.StringType   |
| DATE                 | DataTypes.StringType   |
| DATETIME             | DataTypes.StringType   |
| ARRAY                | 不支持的数据类型   |
| HLL                  | 不支持的数据类型   |
| BITMAP               | 不支持的数据类型   |

当直接使用 DATE 和 DATETIME 数据类型时，StarRocks 使用的底层存储引擎的处理逻辑无法覆盖预期的时间范围。因此，Spark 连接器将 StarRocks 的 DATE 和 DATETIME 数据类型映射到 Spark 的 STRING 数据类型，并生成与从 StarRocks 读取的日期和时间数据相匹配的可读字符串文本。

## 升级 Spark 连接器

### 从版本 1.0.0 升级到版本 1.1.0

- 从 1.1.1 开始，Spark 连接器不提供 `mysql-connector-java`，这是 MySQL 的官方 JDBC 驱动程序，因为使用的 GPL 许可证的限制`mysql-connector-java`。
  但是，Spark 连接器仍然需要连接到 `mysql-connector-java` StarRocks 以获取表元数据，因此需要手动将驱动程序添加到 Spark 类路径中。您可以在 [MySQL 站点](https://dev.mysql.com/downloads/connector/j/) 或 [Maven Central 上](https://repo1.maven.org/maven2/mysql/mysql-connector-java/)找到驱动程序。
  
- 在 1.1.0 版本中，Spark 连接器使用 JDBC 访问 StarRocks，以获取更详细的表信息。因此，必须配置 `starrocks.fe.jdbc.url`。

- 在 1.1.0 版本中，重命名了一些参数。旧参数和新参数现在都保留。对于每对等效参数，您只需要配置其中一个参数，但我们建议您使用新参数，因为旧参数可能已被弃用。
  - `starrocks.fenodes` 重命名为 `starrocks.fe.http.url`。
  - `user` 重命名为 `starrocks.user`。
  - `password` 重命名为 `starrocks.password`。

- 在 1.1.0 版本中，部分数据类型的映射基于 Spark 3.x 进行了调整：
  - `DATE` 在 StarRocks 中映射到 `DataTypes.DateType`（原始为 `DataTypes.StringType`）在 Spark 中。
  - `DATETIME` 在 StarRocks 中映射到 `DataTypes.TimestampType`（原始为 `DataTypes.StringType`）在 Spark 中。

## 例子

以下示例假设您在 StarRocks 集群中创建了一个名为 `test` 的数据库，并且您拥有 `root` 用户的权限。示例中的参数设置基于 Spark 连接器 1.1.0。

### 数据示例

执行以下操作以准备一个示例表：

1. 转到 `test` 数据库并创建一个名为 `score_board` 的表。

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

2. 向 `score_board` 表中插入数据。

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

3. 查询 `score_board` 表。

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

### 使用 Spark SQL 读取数据

1. 在 Spark 目录下执行以下命令，启动 Spark SQL。

   ```Plain
   sh spark-sql
   ```

2. 执行以下命令，在 `test` 数据库中的 `score_board` 表上创建名为 `spark_starrocks` 的临时视图。

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

3. 执行以下命令，从临时视图中读取数据。

   ```SQL
   spark-sql> SELECT * FROM spark_starrocks;
   ```

   Spark 返回以下数据：

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

### 使用 Spark DataFrame 读取数据

1. 在 Spark 目录下执行以下命令，启动 Spark Shell。

   ```Plain
   sh spark-shell
   ```

2. 运行以下命令，在 `test` 数据库的 `score_board` 表上创建名为 `starrocksSparkDF` 的 DataFrame：

   ```Scala
   scala> val starrocksSparkDF = spark.read.format("starrocks")
              .option("starrocks.table.identifier", s"test.score_board")
              .option("starrocks.fe.http.url", s"<fe_host>:<fe_http_port>")
              .option("starrocks.fe.jdbc.url", s"jdbc:mysql://<fe_host>:<fe_query_port>")
              .option("starrocks.user", s"root")
              .option("starrocks.password", s"")
              .load()
   ```

3. 从 DataFrame 中读取数据。例如，如果要读取前 10 行，请运行以下命令：

   ```Scala
   scala> starrocksSparkDF.show(10)
   ```

   Spark 返回以下数据：

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
   > 默认情况下，如果您未指定要读取的行数，Spark 将返回前 20 行。

### 使用 Spark RDD 读取数据

1. 在 Spark 目录中运行以下命令启动 Spark Shell：

   ```Plain
   sh spark-shell
   ```

2. 运行以下命令，在 `test` 数据库的 `score_board` 表上创建名为 `starrocksSparkRDD` 的 RDD：

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

3. 从 RDD 中读取数据。例如，如果要读取前 10 个元素，请运行以下命令：

   ```Scala
   scala> starrocksSparkRDD.take(10)
   ```

   Spark 返回以下数据：

   ```Scala
   res0: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24])
   ```

   要读取整个 RDD，请运行以下命令：

   ```Scala
   scala> starrocksSparkRDD.collect()
   ```

   Spark 返回以下数据：

   ```Scala
   res1: Array[AnyRef] = Array([1, Bob, 21], [2, Stan, 21], [3, Sam, 22], [4, Tony, 22], [5, Alice, 22], [6, Lucy, 23], [7, Polly, 23], [8, Tom, 23], [9, Rose, 24], [10, Jerry, 24], [11, Jason, 24], [12, Lily, 25], [13, Stephen, 25], [14, David, 25], [15, Eddie, 26], [16, Kate, 27], [17, Cathy, 27], [18, Judy, 27], [19, Julia, 28], [20, Robert, 28], [21, Jack, 29])
   ```

## 最佳实践

当您使用 Spark 连接器从 StarRocks 读取数据时，您可以使用 `starrocks.filter.query` 参数指定筛选条件，根据 Spark 对分区、存储桶和前缀索引进行修剪，以降低数据拉取成本。本部分以 Spark DataFrame 为例，演示如何实现这一点。

### 环境设置

| 组件       | 版本                                                      |
| --------------- | ------------------------------------------------------------ |
| Spark           | Spark 2.4.4 和 Scala 2.11.12（OpenJDK 64 位服务器 VM，Java 1.8.0_302） |
| StarRocks（星石）       | 2.2.0                                                        |
| Spark 连接器 | starrocks-spark2_2.11-1.0.0.jar                              |

### 数据示例

执行以下操作以准备示例表：

1. 转到 `test` 数据库并创建一个名为 `mytable` 的表。

   ```SQL
   MySQL [test]> CREATE TABLE `mytable`
   (
       `k` int(11) NULL COMMENT "bucket",
       `b` int(11) NULL COMMENT "",
       `dt` datetime NULL COMMENT "",
       `v` int(11) NULL COMMENT ""
   )
   ENGINE=OLAP
   DUPLICATE KEY(`k`,`b`, `dt`)
   COMMENT "OLAP"
   PARTITION BY RANGE(`dt`)
   (
       PARTITION p202201 VALUES [('2022-01-01 00:00:00'), ('2022-02-01 00:00:00')),
       PARTITION p202202 VALUES [('2022-02-01 00:00:00'), ('2022-03-01 00:00:00')),
       PARTITION p202203 VALUES [('2022-03-01 00:00:00'), ('2022-04-01 00:00:00'))
   )
   DISTRIBUTED BY HASH(`k`)
   PROPERTIES (
       "replication_num" = "3"
   );
   ```

2. 将数据插入到 `mytable` 中。

   ```SQL
   MySQL [test]> INSERT INTO mytable
   VALUES
        (1, 11, '2022-01-02 08:00:00', 111),
        (2, 22, '2022-02-02 08:00:00', 222),
        (3, 33, '2022-03-02 08:00:00', 333);
   ```

3. 查询 `mytable` 表。

   ```SQL
   MySQL [test]> select * from mytable;
   +------+------+---------------------+------+
   | k    | b    | dt                  | v    |
   +------+------+---------------------+------+
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    2 |   22 | 2022-02-02 08:00:00 |  222 |
   |    3 |   33 | 2022-03-02 08:00:00 |  333 |
   +------+------+---------------------+------+
   3 rows in set (0.01 sec)
   ```

### 全表扫描

1. 在 Spark 目录中运行以下命令，创建属于 `mytable` 数据库的名为 `df` 的 DataFrame。

   ```Scala
   scala>  val df = spark.read.format("starrocks")
           .option("starrocks.table.identifier", s"test.mytable")
           .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
           .option("user", s"root")
           .option("password", s"")
           .load()
   ```

2. 查看 StarRocks 集群的 FE 日志文件 **fe.log**，找到读取数据的 SQL 语句。例如：

   ```SQL
   2022-08-09 18:57:38,091 INFO (nioEventLoopGroup-3-10|196) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable`] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. 在 `test` 数据库中，使用 EXPLAIN 获取 SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` 语句的执行计划。

   ```SQL
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable`;
   +-----------------------------------------------------------------------+
   | Explain String                                                        |
   +-----------------------------------------------------------------------+
   | PLAN FRAGMENT 0                                                       |
   |  OUTPUT EXPRS:1: k | 2: b | 3: dt | 4: v                              |
   |   PARTITION: UNPARTITIONED                                            |
   |                                                                       |
   |   RESULT SINK                                                         |
   |                                                                       |
   |   1:EXCHANGE                                                          |
   |                                                                       |
   | PLAN FRAGMENT 1                                                       |
   |  OUTPUT EXPRS:                                                        |
   |   PARTITION: RANDOM                                                   |
   |                                                                       |
   |   STREAM DATA SINK                                                    |
   |     EXCHANGE ID: 01                                                   |
   |     UNPARTITIONED                                                     |
   |                                                                       |
   |   0:OlapScanNode                                                      |
   |      TABLE: mytable                                                   |
   |      PREAGGREGATION: ON                                               |
   |      partitions=3/3                                                   |
   |      rollup: mytable                                                  |
   |      tabletRatio=9/9                                                  |
   |      tabletList=41297,41299,41301,41303,41305,41307,41309,41311,41313 |
   |      cardinality=3                                                    |
   |      avgRowSize=4.0                                                   |
   |      numNodes=0                                                       |
   +-----------------------------------------------------------------------+
   26 rows in set (0.00 sec)
   ```

在此示例中，未执行修剪。因此，Spark 扫描了保存数据的三个分区（如 `partitions=3/3` 所示）和这三个分区中的所有 9 个平板电脑（如 `tabletRatio=9/9` 所示）。

### 分区修剪

1. 在 Spark 目录中运行以下命令，使用 `starrocks.filter.query` 参数指定分区修剪的筛选条件 `dt='2022-01-02 08:00:00'`，创建属于 `mytable` 数据库的名为 `df` 的 DataFrame。

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "dt='2022-01-02 08:00:00'")
          .load()
   ```

2. 查看 StarRocks 集群的 FE 日志文件 **fe.log**，找到读取数据的 SQL 语句。例如：

   ```SQL
   2022-08-09 19:02:31,253 INFO (nioEventLoopGroup-3-14|204) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. 在 `test` 数据库中，使用 EXPLAIN 获取 SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00' 语句的执行计划：

   ```SQL
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00';
   +------------------------------------------------+
   | Explain String                                 |
   +------------------------------------------------+
   | PLAN FRAGMENT 0                                |
   |  OUTPUT EXPRS:1: k | 2: b | 3: dt | 4: v       |
   |   PARTITION: UNPARTITIONED                     |
   |                                                |
   |   RESULT SINK                                  |
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
   |      PREDICATES: 3: dt = '2022-01-02 08:00:00' |
   |      partitions=1/3                            |
   |      rollup: mytable                           |
   |      tabletRatio=3/3                           |
   |      tabletList=41297,41299,41301              |
   |      cardinality=1                             |
   |      avgRowSize=20.0                           |
   |      numNodes=0                                |
   +------------------------------------------------+
   27 rows in set (0.01 sec)
   ```

在此示例中，仅执行了分区剪枝，而未执行存储桶剪枝。因此，Spark 会扫描三个分区中的一个（如`partitions=1/3`所示），以及该分区中的所有平板电脑（如`tabletRatio=3/3`所示）。

### 存储桶剪枝

1. 在Spark目录中运行以下命令，使用`starrocks.filter.query`参数指定`k=1`作为存储桶剪枝的筛选条件，在`test`数据库中的`mytable`表上创建名为`df`的DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

2. 查看StarRocks集群的FE日志文件**fe.log**，找到用于读取数据的SQL语句。例如：

   ```SQL
   2022-08-09 19:04:44,479 INFO (nioEventLoopGroup-3-16|208) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. 在`test`数据库中，使用EXPLAIN获取SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1语句的执行计划：

   ```Scala
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1;
   +------------------------------------------+
   | Explain String                           |
   +------------------------------------------+
   | PLAN FRAGMENT 0                          |
   |  OUTPUT EXPRS:1: k | 2: b | 3: dt | 4: v |
   |   PARTITION: UNPARTITIONED               |
   |                                          |
   |   RESULT SINK                            |
   |                                          |
   |   1:EXCHANGE                             |
   |                                          |
   | PLAN FRAGMENT 1                          |
   |  OUTPUT EXPRS:                           |
   |   PARTITION: RANDOM                      |
   |                                          |
   |   STREAM DATA SINK                       |
   |     EXCHANGE ID: 01                      |
   |     UNPARTITIONED                        |
   |                                          |
   |   0:OlapScanNode                         |
   |      TABLE: mytable                      |
   |      PREAGGREGATION: ON                  |
   |      PREDICATES: 1: k = 1                |
   |      partitions=3/3                      |
   |      rollup: mytable                     |
   |      tabletRatio=3/9                     |
   |      tabletList=41299,41305,41311        |
   |      cardinality=1                       |
   |      avgRowSize=20.0                     |
   |      numNodes=0                          |
   +------------------------------------------+
   27 rows in set (0.01 sec)
   ```

在此示例中，仅执行了存储桶剪枝，而未执行分区剪枝。因此，Spark会扫描包含数据的所有三个分区（如`partitions=3/3`所示），并扫描所有三个平板电脑（如`tabletRatio=3/9`所示），以检索满足这三个分区中的`k = 1`筛选条件的哈希值。

### 分区剪枝和存储桶剪枝

1. 在Spark目录中运行以下命令，使用`starrocks.filter.query`参数指定`k=7`和`dt='2022-01-02 08:00:00'`作为存储桶剪枝和分区剪枝的两个筛选条件，在`test`数据库中的`mytable`表上创建名为`df`的DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"")
          .option("password", s"")
          .option("starrocks.filter.query", "k=7 and dt='2022-01-02 08:00:00'")
          .load()
   ```

2. 查看StarRocks集群的FE日志文件**fe.log**，找到用于读取数据的SQL语句。例如：

   ```SQL
   2022-08-09 19:06:34,939 INFO (nioEventLoopGroup-3-18|212) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] t
   able [mytable]
   ```

3. 在`test`数据库中，使用EXPLAIN获取SELECT `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00'语句的执行计划：

   ```Scala
   MySQL [test]> EXPLAIN select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=7 and dt='2022-01-02 08:00:00';
   +----------------------------------------------------------+
   | Explain String                                           |
   +----------------------------------------------------------+
   | PLAN FRAGMENT 0                                          |
   |  OUTPUT EXPRS:1: k | 2: b | 3: dt | 4: v                 |
   |   PARTITION: RANDOM                                      |
   |                                                          |
   |   RESULT SINK                                            |
   |                                                          |
   |   0:OlapScanNode                                         |
   |      TABLE: mytable                                      |
   |      PREAGGREGATION: ON                                  |
   |      PREDICATES: 1: k = 7, 3: dt = '2022-01-02 08:00:00' |
   |      partitions=1/3                                      |
   |      rollup: mytable                                     |
   |      tabletRatio=1/3                                     |
   |      tabletList=41301                                    |
   |      cardinality=1                                       |
   |      avgRowSize=20.0                                     |
   |      numNodes=0                                          |
   +----------------------------------------------------------+
   17 rows in set (0.00 sec)
   ```

在此示例中，同时执行了分区剪枝和存储桶剪枝。因此，Spark仅扫描三个分区中的一个（如`partitions=1/3`所示），并且只扫描一个平板电脑（如`tabletRatio=1/3`所示）。

### 前缀索引过滤

1. 将更多数据记录插入到属于`test`数据库的`mytable`表的分区中：

   ```Scala
   MySQL [test]> INSERT INTO mytable
   VALUES
       (1, 11, "2022-01-02 08:00:00", 111), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333);
   ```

2. 查询`mytable`表：

   ```Scala
   MySQL [test]> SELECT * FROM mytable;
   +------+------+---------------------+------+
   | k    | b    | dt                  | v    |
   +------+------+---------------------+------+
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    1 |   11 | 2022-01-02 08:00:00 |  111 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    3 |   33 | 2022-01-02 08:00:00 |  333 |
   |    2 |   22 | 2022-02-02 08:00:00 |  222 |
   |    3 |   33 | 2022-03-02 08:00:00 |  333 |
   +------+------+---------------------+------+
   7 rows in set (0.01 sec)
   ```

3. 在Spark目录中运行以下命令，使用`starrocks.filter.query`参数指定`k=1`作为前缀索引过滤的筛选条件，在`test`数据库中的`mytable`表上创建名为`df`的DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", s"test.mytable")
          .option("starrocks.fenodes", s"<fe_host>:<fe_http_port>")
          .option("user", s"root")
          .option("password", s"")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

4. 在`test`数据库中，将`is_report_success`设置为`true`以启用配置文件报告：

   ```SQL
   MySQL [test]> SET is_report_success = true;
   Query OK, 0 rows affected (0.00 sec)
   ```

5. 使用浏览器打开`http://<fe_host>:<http_http_port>/query`页面，查看SELECT * FROM mytable where k=1语句的配置文件。例如：

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

在这个例子中，筛选条件 `k = 1` 可以命中前缀索引。因此，Spark可以过滤掉三行（如 `ShortKeyFilterRows: 3`）。