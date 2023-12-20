---
displayed_sidebar: English
---

# 使用 Flink 连接器从 StarRocks 读取数据

StarRocks 提供了一个自研的连接器，名为 StarRocks Connector for Apache Flink®（简称 Flink 连接器），以帮助您利用 Flink 从 StarRocks 集群批量读取数据。

Flink 连接器支持两种读取方法：Flink SQL 和 Flink DataStream。推荐使用 Flink SQL。

> **注意**
> Flink 连接器也支持将 Flink 读取的数据写入另一个 StarRocks 集群或存储系统。详见[从 Apache Flink® 持续加载数据](../loading/Flink-connector-starrocks.md)。

## 背景信息

与 Flink 提供的 JDBC 连接器不同，StarRocks 的 Flink 连接器支持并行从 StarRocks 集群的多个 BE 读取数据，极大地加速了读取任务。以下比较展示了两种连接器的实现差异。

- StarRocks 的 Flink 连接器

  使用 StarRocks 的 Flink 连接器时，Flink 首先从负责的 FE 获取查询计划，然后将获取的查询计划作为参数分发给所有涉及的 BE，最终获取 BE 返回的数据。

  ![- StarRocks 的 Flink 连接器](../assets/5.3.2-1.png)

- Flink 的 JDBC 连接器

  使用 Flink 的 JDBC 连接器时，Flink 只能一次从单个 FE 读取数据。数据读取速度较慢。

  ![Flink 的 JDBC 连接器](../assets/5.3.2-2.png)

## 版本要求

| 连接器 | Flink | StarRocks | Java | Scala |
| --- | --- | --- | --- | --- |
| 1.2.9 | 1.15, 1.16, 1.17, 1.18 | 2.1 及更高版本 | 8 | 2.11, 2.12 |
| 1.2.8 | 1.13, 1.14, 1.15, 1.16, 1.17 | 2.1 及更高版本 | 8 | 2.11, 2.12 |
| 1.2.7 | 1.11, 1.12, 1.13, 1.14, 1.15 | 2.1 及更高版本 | 8 | 2.11, 2.12 |

## 先决条件

Flink 已经部署。如果尚未部署 Flink，请按照以下步骤进行部署：

1. 在您的操作系统中安装 Java 8 或 Java 11，以确保 Flink 能够正常运行。您可以使用以下命令检查 Java 安装的版本：

   ```SQL
   java -version
   ```

   例如，如果返回以下信息，则说明已安装 Java 8：

   ```SQL
   openjdk version "1.8.0_322"
   OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
   OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
   ```

2. 下载并解压您选择的 [Flink 包](https://flink.apache.org/downloads.html)。

      > **注意**
      > 我们推荐您使用 Flink v1.14 或更高版本。支持的最低 Flink 版本是 v1.11。

   ```SQL
   # 下载 Flink 包。
   wget https://dlcdn.apache.org/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
   # 解压 Flink 包。
   tar -xzf flink-1.14.5-bin-scala_2.11.tgz
   # 进入 Flink 目录。
   cd flink-1.14.5
   ```

3. 启动您的 Flink 集群。

   ```SQL
   # 启动 Flink 集群。
   ./bin/start-cluster.sh
   
   # 当显示以下信息时，您的 Flink 集群已成功启动：
   Starting cluster.
   Starting standalonesession daemon on host.
   Starting taskexecutor daemon on host.
   ```

您也可以按照 [Flink 文档](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/) 提供的说明来部署 Flink。

## 在您开始之前

按照以下步骤部署 Flink 连接器：

1. 选择并下载与您使用的 Flink 版本匹配的 [flink-connector-starrocks](https://github.com/StarRocks/flink-connector-starrocks/releases) JAR 包。

      > **注意**
      > 我们建议您下载版本为 1.2.x 或更高的 Flink 连接器包，并确保所匹配的 Flink 版本的前两位数字与您使用的 Flink 版本相同。例如，如果您使用 Flink v1.14.x，则可以下载 `flink-connector-starrocks-1.2.4_flink-1.14_x.yy.jar`。

2. 如果需要进行代码调试，请根据您的业务需求编译 Flink 连接器包。

3. 将您下载或编译的 Flink 连接器包放入 Flink 的 `lib` 目录中。

4. 重新启动您的 Flink 集群。

## 参数

### 通用参数

以下参数适用于 Flink SQL 和 Flink DataStream 读取方法。

| 参数 | 必需 | 数据类型 | 说明 |
| --- | --- | --- | --- |
| connector | 是 | STRING | 您想要使用的连接器类型。值设为 `starrocks`。|
| scan-url | 是 | STRING | 用于从 Web 服务器连接 FE 的地址。格式为 `<fe_host>:<fe_http_port>`。默认端口是 `8030`。您可以指定多个地址，地址之间用逗号（,）分隔。例如：`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。|
| jdbc-url | 是 | STRING | 用于连接 FE 的 MySQL 客户端的地址。格式为 `jdbc:mysql://<fe_host>:<fe_query_port>`。默认端口号是 `9030`。|
| username | 是 | STRING | 您 StarRocks 集群账户的用户名。账户必须对您想要读取的 StarRocks 表有读权限。参见[用户权限](../administration/User_privilege.md)。|
| password | 是 | STRING | 您 StarRocks 集群账户的密码。|
| database-name | 是 | STRING | 您想要读取的 StarRocks 表所属的 StarRocks 数据库名称。|
| table-name | 是 | STRING | 您想要读取的 StarRocks 表的名称。|
| scan.connect.timeout-ms | 否 | STRING | Flink 连接器连接到 StarRocks 集群的最大超时时间。单位为毫秒。默认值是 `1000`。如果建立连接的时间超过此限制，读取任务将失败。|
| scan.params.keep-alive-min | 否 | STRING | 读取任务保持活动状态的最大时间。通过轮询机制定期检查保活时间。单位为分钟。默认值是 `10`。建议您将此参数设置为大于或等于 `5` 的值。|
| scan.params.query-timeout-s | 否 | STRING | 读取任务超时的最大时间。在任务执行期间检查超时持续时间。单位为秒。默认值是 `600`。如果在该时间段后没有返回读取结果，读取任务将停止。|
| scan.params.mem-limit-byte | 否 | STRING | 每个 BE 上每个查询允许的最大内存量。单位为字节。默认值是 `1073741824`，相当于 1 GB。|
| scan.max-retries | 否 | STRING | 读取任务失败时可以重试的最大次数。默认值是 `1`。如果读取任务的重试次数超过此限制，读取任务将返回错误。|

### Flink DataStream 参数

以下参数仅适用于 Flink DataStream 读取方法。

| 参数 | 必需 | 数据类型 | 描述 |
| --- | --- | --- | --- |
| scan.columns | 否 | STRING | 您想要读取的列。可以指定多个列，列之间用逗号（,）分隔。|
| scan.filter | 否 | STRING | 您想要基于其过滤数据的过滤条件。|

假设在 Flink 中创建了一个包含三列 `c1`、`c2`、`c3` 的表。要读取此 Flink 表中 `c1` 列值等于 `100` 的行，您可以指定两个过滤条件 `"scan.columns", "c1"` 和 `"scan.filter", "c1 = 100"`。

## StarRocks 与 Flink 之间的数据类型映射

```
以下数据类型映射仅适用于 Flink 从 StarRocks 读取数据。有关 Flink 将数据写入 StarRocks 所使用的数据类型映射，请参阅[从 Apache Flink® 持续加载数据](../loading/Flink-connector-starrocks.md)。

|StarRocks|Flink|
|---|---|
|NULL|NULL|
|BOOLEAN|BOOLEAN|
|TINYINT|TINYINT|
|SMALLINT|SMALLINT|
|INT|INT|
|BIGINT|BIGINT|
|LARGEINT|STRING|
|FLOAT|FLOAT|
|DOUBLE|DOUBLE|
|DATE|DATE|
|DATETIME|TIMESTAMP|
|DECIMAL|DECIMAL|
|DECIMALV2|DECIMAL|
|DECIMAL32|DECIMAL|
|DECIMAL64|DECIMAL|
|DECIMAL128|DECIMAL|
|CHAR|CHAR|
|VARCHAR|STRING|

## 示例

以下示例假设您已在 StarRocks 集群中创建了名为 `test` 的数据库，并且拥有用户 `root` 的权限。

> **注意**
> 如果读取任务失败，您必须重新创建它。

### 数据示例

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
   PROPERTIES
   (
       "replication_num" = "3"
   );
   ```

2. 将数据插入 `score_board` 表中。

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
   21 rows in set (0.00 sec)
   ```

### 使用 Flink SQL 读取数据

1. 在您的 Flink 集群中，根据源 StarRocks 表（本例中为 `score_board`）的架构创建一个名为 `flink_test` 的表。在建表命令中，需要配置读任务属性，包括 Flink 连接器、源 StarRocks 数据库和源 StarRocks 表的信息。

   ```SQL
   CREATE TABLE flink_test
   (
       `id` INT,
       `name` STRING,
       `score` INT
   )
   WITH
   (
       'connector'='starrocks',
       'scan-url'='192.168.xxx.xxx:8030',
       'jdbc-url'='jdbc:mysql://192.168.xxx.xxx:9030',
       'username'='xxxxxx',
       'password'='xxxxxx',
       'database-name'='test',
       'table-name'='score_board'
   );
   ```

2. 使用 SELECT 从 StarRocks 读取数据。

   ```SQL
   SELECT id, name FROM flink_test WHERE score > 20;
   ```

使用 Flink SQL 读取数据时，需要注意以下几点：

- 您只能使用 `SELECT ... FROM <table_name> WHERE ...` 等 SQL 语句从 StarRocks 读取数据。在所有聚合函数中，仅支持 `count`。
- 支持谓词下推。例如，如果您的查询包含过滤条件 `char_1 <> 'A' AND int_1 = -126`，则该过滤条件将被下推到 Flink 连接器，并在查询运行之前转换为 StarRocks 可以执行的语句。您不需要执行额外的配置。
- 不支持 LIMIT 语句。
- StarRocks 不支持检查点机制。因此，如果读取任务失败，则无法保证数据的一致性。

### 使用 Flink DataStream 读取数据

1. 在 `pom.xml` 文件中添加以下依赖项：

   ```SQL
   <dependency>
       <groupId>com.starrocks</groupId>
       <artifactId>flink-connector-starrocks</artifactId>
       <!-- for Apache Flink® 1.15 -->
       <version>x.x.x_flink-1.15</version>
       <!-- for Apache Flink® 1.14 -->
       <version>x.x.x_flink-1.14_2.11</version>
       <version>x.x.x_flink-1.14_2.12</version>
       <!-- for Apache Flink® 1.13 -->
       <version>x.x.x_flink-1.13_2.11</version>
       <version>x.x.x_flink-1.13_2.12</version>
       <!-- for Apache Flink® 1.12 -->
       <version>x.x.x_flink-1.12_2.11</version>
       <version>x.x.x_flink-1.12_2.12</version>
       <!-- for Apache Flink® 1.11 -->
       <version>x.x.x_flink-1.11_2.11</version>
       <version>x.x.x_flink-1.11_2.12</version>
   </dependency>
   ```

   您必须将上述代码示例中的 `x.x.x` 替换为您正在使用的最新 Flink 连接器版本。请参阅[版本信息](https://search.maven.org/search?q=g:com.starrocks)。

2. 调用 Flink 连接器从 StarRocks 读取数据：

   ```Java
   import com.starrocks.connector.flink.StarRocksSource;
   import com.starrocks.connector.flink.table.source.StarRocksSourceOptions;
   import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
   import org.apache.flink.table.api.DataTypes;
   import org.apache.flink.table.api.TableSchema;
   
   public class StarRocksSourceApp {
           public static void main(String[] args) throws Exception {
               StarRocksSourceOptions options = StarRocksSourceOptions.builder()
                      .withProperty("scan-url", "192.168.xxx.xxx:8030")
                      .withProperty("jdbc-url", "jdbc:mysql://192.168.xxx.xxx:9030")
                      .withProperty("username", "root")
                      .withProperty("password", "")
                      .withProperty("table-name", "score_board")
                      .withProperty("database-name", "test")
                      .build();
               TableSchema tableSchema = TableSchema.builder()
                      .field("id", DataTypes.INT())
                      .field("name", DataTypes.STRING())
                      .field("score", DataTypes.INT())
                      .build();
               StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
               env.addSource(StarRocksSource.source(tableSchema, options)).setParallelism(5).print();
               env.execute("StarRocks Flink source");
           }
   
       }
   ```

## 接下来是什么

在 Flink 成功从 StarRocks 读取数据后，您可以使用 [Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui) 来监控读取任务。例如，您可以在 WebUI 的 **Metrics** 页面上查看 `totalScannedRows` 指标，以获取成功读取的行数。您还可以使用 Flink SQL 对您已读取的数据执行计算，例如联接操作。