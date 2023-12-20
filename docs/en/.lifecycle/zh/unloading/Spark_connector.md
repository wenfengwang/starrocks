---
displayed_sidebar: English
---


|starrocks.fe.http.url|无|FE 的 HTTP IP 地址。Spark 连接器 1.1.0 及以上版本支持此参数。该参数等同于 `starrocks.fenodes`。您只需配置其中之一。在 Spark 连接器 1.1.0 及更高版本中，我们建议您使用 `starrocks.fe.http.url`，因为 `starrocks.fenodes` 可能会被弃用。|
|starrocks.fe.jdbc.url|无|用于连接 FE 的 MySQL 服务器的地址。格式：`jdbc:mysql://<fe_host>:<fe_query_port>`。<br />**注意**<br />在 Spark 连接器 1.1.0 及更高版本中，此参数是必需的。|
|user|无|您的 StarRocks 集群账户的用户名。用户需要 StarRocks 表的 SELECT 权限。|
|starrocks.user|无|您的 StarRocks 集群账户的用户名。Spark 连接器 1.1.0 及以上版本支持此参数。该参数等同于 `user`。您只需配置其中之一。在 Spark 连接器 1.1.0 及更高版本中，我们建议您使用 `starrocks.user`，因为 `user` 可能会被弃用。|
|password|无|您的 StarRocks 集群账户的密码。|
|starrocks.password|无|您的 StarRocks 集群账户的密码。Spark 连接器 1.1.0 及以上版本支持此参数。该参数等同于 `password`。您只需配置其中之一。在 Spark 连接器 1.1.0 及更高版本中，我们建议您使用 `starrocks.password`，因为 `password` 可能会被弃用。|
|starrocks.filter.query.in.max.count|100|谓词下推期间 IN 表达式支持的最大值数。如果 IN 表达式中指定的值数量超过此限制，则 Spark 上将处理 IN 表达式中指定的过滤条件。|

### Spark RDD 的参数

以下参数仅适用于 Spark RDD 读取方式。

|参数|默认值|说明|
|---|---|---|
|starrocks.request.auth.user|无|您的 StarRocks 集群账户的用户名。|
|starrocks.request.auth.password|无|您的 StarRocks 集群账户的密码。|
|starrocks.read.field|无|您希望从中读取数据的 StarRocks 表列。您可以指定多个列，列之间必须用逗号 (,) 分隔。|

## StarRocks 和 Spark 之间的数据类型映射

### Spark 连接器 1.1.0 及更高版本

|StarRocks 数据类型|Spark 数据类型|
|---|---|
|BOOLEAN|DataTypes.BooleanType|
|TINYINT|DataTypes.ByteType|
|SMALLINT|DataTypes.ShortType|
|INT|DataTypes.IntegerType|
|BIGINT|DataTypes.LongType|
|LARGEINT|DataTypes.StringType|
|FLOAT|DataTypes.FloatType|
|DOUBLE|DataTypes.DoubleType|
|DECIMAL|DecimalType|
|CHAR|DataTypes.StringType|
|VARCHAR|DataTypes.StringType|
|STRING|DataTypes.StringType|
|DATE|DataTypes.DateType|
|DATETIME|DataTypes.TimestampType|
|ARRAY|不支持的数据类型|
|HLL|不支持的数据类型|
|BITMAP|不支持的数据类型|

### Spark 连接器 1.0.0

|StarRocks 数据类型|Spark 数据类型|
|---|---|
|BOOLEAN|DataTypes.BooleanType|
|TINYINT|DataTypes.ByteType|
|SMALLINT|DataTypes.ShortType|
|INT|DataTypes.IntegerType|
|BIGINT|DataTypes.LongType|
|LARGEINT|DataTypes.StringType|
|FLOAT|DataTypes.FloatType|
|DOUBLE|DataTypes.DoubleType|
|DECIMAL|DecimalType|
|CHAR|DataTypes.StringType|
|VARCHAR|DataTypes.StringType|
|DATE|DataTypes.StringType|
|DATETIME|DataTypes.StringType|
|ARRAY|不支持的数据类型|
|HLL|不支持的数据类型|
|BITMAP|不支持的数据类型|

StarRocks 使用的底层存储引擎的处理逻辑在直接使用 DATE 和 DATETIME 数据类型时无法覆盖预期的时间范围。因此，Spark 连接器将 StarRocks 中的 DATE 和 DATETIME 数据类型映射为 Spark 中的 STRING 数据类型，并生成与从 StarRocks 读取的日期和时间数据匹配的可读字符串文本。

## 升级 Spark 连接器

### 从 1.0.0 版本升级到 1.1.0 版本

- 从 1.1.1 开始，由于 `mysql-connector-java` 使用的 GPL 许可证的限制，Spark 连接器不再提供 MySQL 的官方 JDBC 驱动程序 `mysql-connector-java`。但是，Spark 连接器仍然需要 `mysql-connector-java` 来连接到 StarRocks 获取表元数据，因此您需要手动将驱动程序添加到 Spark 类路径中。您可以在 [MySQL 网站](https://dev.mysql.com/downloads/connector/j/) 或 [Maven Central](https://repo1.maven.org/maven2/mysql/mysql-connector-java/) 上找到该驱动程序。

- 在 1.1.0 版本中，Spark 连接器使用 JDBC 访问 StarRocks 以获取更详细的表信息。因此，您必须配置 `starrocks.fe.jdbc.url`。

- 在 1.1.0 版本中，一些参数被重命名。目前新旧参数均保留。对于每一对等效参数，您只需配置其中一个，但我们建议您使用新参数，因为旧参数可能会被弃用。
  - `starrocks.fenodes` 更名为 `starrocks.fe.http.url`。
  - `user` 更名为 `starrocks.user`。
  - `password` 更名为 `starrocks.password`。

- 在 1.1.0 版本中，基于 Spark 3.x 调整了部分数据类型的映射：
  - StarRocks 中的 DATE 映射为 Spark 中的 DataTypes.DateType（原为 DataTypes.StringType）。
  - StarRocks 中的 DATETIME 映射为 Spark 中的 DataTypes.TimestampType（原为 DataTypes.StringType）。

## 示例

以下示例假设您已在 StarRocks 集群中创建了名为 `test` 的数据库，并且拥有 `root` 用户的权限。示例中的参数设置基于 Spark Connector 1.1.0。

### 数据示例

准备样例表的步骤如下：

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

2. 将数据插入到 `score_board` 表中。

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
   21 行在集 (0.01 秒)
   ```

### 使用 Spark SQL 读取数据

1. 在 Spark 目录下执行以下命令启动 Spark SQL：

   ```Plain
   sh spark-sql
   ```

2. 执行以下命令，在 `test` 数据库的 `score_board` 表上创建一个名为 `spark_starrocks` 的临时视图：

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

3. 执行以下命令从临时视图中读取数据：

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
   10       Jerry       24
   11       Jason       24
   12       Lily        25
   13       Stephen     25
   14       David       25
   15       Eddie       26
   16       Kate        27
   17       Cathy       27
   18       Judy        27
   19       Julia       28
   20       Robert      28
   21       Jack        29
   耗时：1.883 秒，获取 21 行(s)
   22/08/09 15:29:36 INFO thriftserver.SparkSQLCLIDriver: 耗时：1.883 秒，获取 21 行(s)
   ```

### 使用 Spark DataFrame 读取数据

1. 在 Spark 目录下执行以下命令启动 Spark Shell：

   ```Plain
   sh spark-shell
   ```

2. 运行以下命令在 `test` 数据库的 `score_board` 表上创建一个名为 `starrocksSparkDF` 的 DataFrame：

   ```Scala
   scala> val starrocksSparkDF = spark.read.format("starrocks")
              .option("starrocks.table.identifier", "test.score_board")
              .option("starrocks.fe.http.url", "<fe_host>:<fe_http_port>")
              .option("starrocks.fe.jdbc.url", "jdbc:mysql://<fe_host>:<fe_query_port>")
              .option("starrocks.user", "root")
              .option("starrocks.password", "")
              .load()
   ```
```Scala
              .option("starrocks.fe.http.url", s"<fe_host>:<fe_http_port>")
              .option("starrocks.fe.jdbc.url", s"jdbc:mysql://<fe_host>:<fe_query_port>")
              .option("starrocks.user", "root")
              .option("starrocks.password", "")
              .load()
   ```

3. 从 DataFrame 中读取数据。例如，如果您想读取前 10 行数据，可以运行以下命令：

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
   仅显示前 10 行
   ```

      > **注意**
      > 默认情况下，如果您没有指定要读取的行数，Spark 会返回前 20 行数据。

### 使用 Spark RDD 读取数据

1. 在 Spark 目录下运行以下命令以启动 Spark Shell：

   ```Plain
   sh spark-shell
   ```

2. 执行以下命令，在 `test` 数据库的 `score_board` 表上创建一个名为 `starrocksSparkRDD` 的 RDD。

   ```Scala
   scala> import com.starrocks.connector.spark._
   scala> val starrocksSparkRDD = sc.starrocksRDD(
              tableIdentifier = Some("test.score_board"),
              cfg = Some(Map(
                  "starrocks.fenodes" -> "<fe_host>:<fe_http_port>",
                  "starrocks.request.auth.user" -> "root",
                  "starrocks.request.auth.password" -> ""
              ))
              )
   ```

3. 从 RDD 中读取数据。例如，如果您想读取前 10 个元素，可以运行以下命令：

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

当您使用 Spark 连接器从 StarRocks 读取数据时，您可以使用 `starrocks.filter.query` 参数指定基于 Spark 修剪分区、桶和前缀索引的过滤条件，以减少数据拉取的成本。本节以 Spark DataFrame 为例，展示如何实现这一点。

### 环境设置

| 组件       | 版本                                                      |
| ---------- | --------------------------------------------------------- |
| Spark      | Spark 2.4.4 和 Scala 2.11.12（OpenJDK 64 位服务器 VM，Java 1.8.0_302） |
| StarRocks  | 2.2.0                                                     |
| Spark 连接器 | starrocks-spark2_2.11-1.0.0.jar                           |

### 数据示例

准备样例表的步骤如下：

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

2. 将数据插入 `mytable`。

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
   3 行在集 (0.01 秒)
   ```

### 全表扫描

1. 在 Spark 目录中运行以下命令，在 `test` 数据库的 `mytable` 表上创建一个名为 `df` 的 DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
           .option("starrocks.table.identifier", "test.mytable")
           .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
           .option("user", "root")
           .option("password", "")
           .load()
   ```

2. 查看 StarRocks 集群的 FE 日志文件 **fe.log**，找到执行读取数据的 SQL 语句。例如：

   ```SQL
   2022-08-09 18:57:38,091 INFO (nioEventLoopGroup-3-10|196) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable`] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. 在 `test` 数据库中，使用 EXPLAIN 获取 `SELECT k,b,dt,v FROM test.mytable` 语句的执行计划：

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
   26 行在集 (0.00 秒)
   ```

在此示例中，没有执行修剪。因此，Spark 扫描了保存数据的所有三个分区（如 `partitions=3/3` 所示），并扫描了这三个分区中的所有 9 个 tablet（如 `tabletRatio=9/9` 所示）。

### 分区修剪

1. 执行以下命令，在 Spark 目录下创建一个名为 `df` 的 DataFrame，该 DataFrame 属于 `test` 数据库的 `mytable` 表，并通过 `starrocks.filter.query` 参数指定分区修剪的过滤条件 `dt='2022-01-02 08:00:00'`：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
          .option("user", "root")
          .option("password", "")
          .option("starrocks.filter.query", "dt='2022-01-02 08:00:00'")
          .load()
   ```

2. 查看 StarRocks 集群的 FE 日志文件 **fe.log**，找到执行读取数据的 SQL 语句。例如：

   ```SQL
   2022-08-09 19:02:31,253 INFO (nioEventLoopGroup-3-14|204) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. 在 `test` 数据库中，使用 EXPLAIN 获取 `SELECT k,b,dt,v FROM test.mytable WHERE dt='2022-01-02 08:00:00'` 语句的执行计划：

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
   ```
```markdown
   |      tabletList=41297,41299,41301              |
   |      cardinality=1                             |
   |      avgRowSize=20.0                           |
   |      numNodes=0                                |
   +------------------------------------------------+
   27 rows in set (0.01 sec)
   ```

在本例中，仅进行了分区剪枝而没有进行桶剪枝。因此，Spark 会扫描其中一个分区（如 `partitions=1/3` 所示），以及该分区中的所有 tablet（如 `tabletRatio=3/3` 所示）。

### 桶剪枝

1. 在 Spark 目录下执行以下命令，使用 `starrocks.filter.query` 参数指定桶剪枝的过滤条件 `k=1`，在 `test` 数据库的 `mytable` 表上创建一个名为 `df` 的 DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
          .option("user", "root")
          .option("password", "")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

2. 查看 StarRocks 集群的 FE 日志文件 `fe.log`，找到执行读取数据的 SQL 语句。示例：

   ```SQL
   2022-08-09 19:04:44,479 INFO (nioEventLoopGroup-3-16|208) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [select `k`,`b`,`dt`,`v` from `test`.`mytable` where k=1] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. 在 `test` 数据库中，使用 EXPLAIN 获取 `SELECT k,b,dt,v FROM test.mytable WHERE k=1` 语句的执行计划：

   ```Scala
   MySQL [test]> EXPLAIN SELECT `k`,`b`,`dt`,`v` FROM `test`.`mytable` WHERE k=1;
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

在本例中，仅进行了桶剪枝而没有进行分区剪枝。因此，Spark 会扫描所有三个分区（如 `partitions=3/3` 所示），并扫描这些分区中满足 `k = 1` 过滤条件的所有 tablet（如 `tabletRatio=3/9` 所示）。

### 分区剪枝和桶剪枝

1. 在 Spark 目录下执行以下命令，使用 `starrocks.filter.query` 参数指定桶剪枝和分区剪枝的两个过滤条件 `k=7` 和 `dt='2022-01-02 08:00:00'`，在 `test` 数据库的 `mytable` 表上创建一个名为 `df` 的 DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
          .option("user", "")
          .option("password", "")
          .option("starrocks.filter.query", "k=7 AND dt='2022-01-02 08:00:00'")
          .load()
   ```

2. 查看 StarRocks 集群的 FE 日志文件 `fe.log`，找到执行读取数据的 SQL 语句。示例：

   ```SQL
   2022-08-09 19:06:34,939 INFO (nioEventLoopGroup-3-18|212) [TableQueryPlanAction.executeWithoutPassword():126] receive SQL statement [SELECT `k`,`b`,`dt`,`v` FROM `test`.`mytable` WHERE k=7 AND dt='2022-01-02 08:00:00'] from external service [ user ['root'@'%']] for database [test] table [mytable]
   ```

3. 在 `test` 数据库中，使用 EXPLAIN 获取 `SELECT k,b,dt,v FROM test.mytable WHERE k=7 AND dt='2022-01-02 08:00:00'` 语句的执行计划：

   ```Scala
   MySQL [test]> EXPLAIN SELECT `k`,`b`,`dt`,`v` FROM `test`.`mytable` WHERE k=7 AND dt='2022-01-02 08:00:00';
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

在这个例子中，同时进行了分区剪枝和桶剪枝。因此，Spark 仅扫描其中一个分区（如 `partitions=1/3` 所示），并且仅扫描该分区中的一个 tablet（如 `tabletRatio=1/3` 所示）。

### 前缀索引过滤

1. 向 `test` 数据库的 `mytable` 表的一个分区中插入更多数据记录：

   ```SQL
   MySQL [test]> INSERT INTO mytable
   VALUES
       (1, 11, "2022-01-02 08:00:00", 111), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333), 
       (3, 33, "2022-01-02 08:00:00", 333);
   ```

2. 查询 `mytable` 表：

   ```SQL
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

3. 在 Spark 目录下执行以下命令，使用 `starrocks.filter.query` 参数指定过滤条件 `k=1` 进行前缀索引过滤，在 `test` 数据库的 `mytable` 表上创建一个名为 `df` 的 DataFrame：

   ```Scala
   scala> val df = spark.read.format("starrocks")
          .option("starrocks.table.identifier", "test.mytable")
          .option("starrocks.fenodes", "<fe_host>:<fe_http_port>")
          .option("user", "root")
          .option("password", "")
          .option("starrocks.filter.query", "k=1")
          .load()
   ```

4. 在 `test` 数据库中，将 `is_report_success` 设置为 `true` 以启用配置文件报告：

   ```SQL
   MySQL [test]> SET is_report_success = true;
   Query OK, 0 rows affected (0.00 sec)
   ```

5. 使用浏览器打开 `http://<fe_host>:<http_http_port>/query` 页面，查看 `SELECT * FROM mytable WHERE k=1` 语句的简介。示例：

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
   ```
```
```markdown
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

在本例中，过滤条件 `k=1` 能够命中前缀索引。因此，Spark 可以过滤掉三行（如 `ShortKeyFilterRows: 3` 所示）。
```