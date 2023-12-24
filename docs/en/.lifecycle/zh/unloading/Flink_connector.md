---
displayed_sidebar: English
---

# 使用 Flink 连接器从 StarRocks 读取数据

StarRocks 提供了一款自研的连接器，名为 StarRocks Connector for Apache Flink（简称 Flink Connector），可帮助您通过 Flink 从 StarRocks 集群中批量读取数据。

Flink 连接器支持两种读取方法：Flink SQL 和 Flink DataStream。建议使用 Flink SQL。

> **注意**
>
> Flink 连接器还支持将 Flink 读取的数据写入另一个 StarRocks 集群或存储系统。请参阅[从 Apache Flink® 持续加载数据](../loading/Flink-connector-starrocks.md)。

## 背景信息

与 Flink提供的JDBC连接器不同，StarRocks的Flink连接器支持并行从StarRocks集群的多个BE中读取数据，极大加速了读取任务。以下比较显示了这两种连接器在实现上的差异。

- StarRocks的Flink连接器

  使用StarRocks的Flink连接器，Flink可以先从负责的FE获取查询计划，然后将获取到的查询计划作为参数分发给所有涉及的BE，最后获取BE返回的数据。

  ![- StarRocks的Flink连接器](../assets/5.3.2-1.png)

- Flink的JDBC连接器

  使用Flink的JDBC连接器，Flink只能从单个FE中逐个读取数据，读取速度较慢。

  ![Flink的JDBC连接器](../assets/5.3.2-2.png)

## 版本要求

| 连接器 | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.9 | 1.15,1.16,1.17,1.18 | 2.1及更高版本| 8 | 2.11,2.12 |
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1及更高版本| 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1及更高版本| 8    | 2.11,2.12 |

## 先决条件

已部署Flink。如果尚未部署Flink，请按照以下步骤进行部署：

1. 在操作系统中安装Java 8或Java 11，以确保Flink能够正常运行。您可以使用以下命令检查Java安装的版本：

   ```SQL
   java -version
   ```

   例如，如果返回以下信息，则表示已安装Java 8：

   ```SQL
   openjdk version "1.8.0_322"
   OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
   OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
   ```

2. 下载并解压缩[您选择的](https://flink.apache.org/downloads.html)Flink包。

   > **注意**
   >
   > 建议您使用Flink v1.14或更高版本。支持的最低Flink版本为v1.11。

   ```SQL
   # 下载Flink包。
   wget https://dlcdn.apache.org/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
   # 解压缩Flink包。
   tar -xzf flink-1.14.5-bin-scala_2.11.tgz
   # 进入Flink目录。
   cd flink-1.14.5
   ```

3. 启动Flink集群。

   ```SQL
   # 启动您的Flink集群。
   ./bin/start-cluster.sh
         
   # 当显示以下信息时，您的Flink集群已成功启动：
   Starting cluster.
   Starting standalonesession daemon on host.
   Starting taskexecutor daemon on host.
   ```

您也可以按照[Flink文档](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)中提供的说明部署Flink。

## 准备工作

按照以下步骤部署Flink连接器：

1. 选择并下载[与当前Flink版本匹配的](https://github.com/StarRocks/flink-connector-starrocks/releases)flink-connector-starrocks JAR包。

   > **通知**
   >
   > 建议您下载版本为1.2.x及以上且Flink版本与Flink版本前两位数字相同的Flink连接器包。例如，如果您使用Flink v1.14.x，则可以下载`flink-connector-starrocks-1.2.4_flink-1.14_x.yy.jar`。

2. 如果需要进行代码调试，请编译Flink连接器包以满足您的业务需求。

3. 将您下载或编译的Flink连接器包放入Flink的`lib`目录。

4. 重启Flink集群。

## 参数

### 常用参数

以下参数适用于Flink SQL和Flink DataStream读取方法。

| 参数                   | 必填 | 数据类型 | 描述                                                  |
| --------------------------- | -------- | --------- | ------------------------------------------------------------ |
| 连接器                   | 是      | 字符串    | 要用于读取数据的连接器类型。将值设置为 `starrocks`。                                |
| scan-url                    | 是      | 字符串    | 用于连接FE的Web服务器的地址。格式：`<fe_host>:<fe_http_port>`。默认端口为`8030`。您可以指定多个地址，必须用逗号（,）分隔。示例：`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| jdbc-url                    | 是      | 字符串    | 用于连接FE的MySQL客户端的地址。格式：`jdbc:mysql://<fe_host>:<fe_query_port>`。默认端口号为`9030`。 |
| 用户名                    | 是      | 字符串    | 您的StarRocks集群账号的用户名。该账号必须对要读取的StarRocks表具有读取权限。请参阅[用户权限](../administration/User_privilege.md)。 |
| 密码                    | 是      | 字符串    | 您的StarRocks集群账号密码。              |
| 数据库名称               | 是      | 字符串    | 要读取的StarRocks表所属的StarRocks数据库名称。 |
| 表名                  | 是      | 字符串    | 要读取的StarRocks表的名称。            |
| scan.connect.timeout-ms     | 否       | 字符串    | Flink连接器与StarRocks集群连接的最长超时时间。单位：毫秒。默认值：`1000`。如果建立连接的时间超过此限制，读取任务将失败。 |
| scan.params.keep-alive-min  | 否       | 字符串    | 读取任务保持活动状态的最长时间。使用轮询机制定期检查保持活动时间。单位：分钟。默认值：`10`。建议将此参数设置为大于或等于`5`的值。 |
| scan.params.query-timeout-s | 否       | 字符串    | 读取任务的最长超时时间。在任务执行期间检查超时持续时间。单位：秒。默认值：`600`。如果超过该时间后仍未返回读取结果，则读取任务将停止。 |
| scan.params.mem-limit-byte  | 否       | 字符串    | 每个BE上每个查询允许的最大内存量。单位：字节。默认值：`1073741824`，相当于1GB。 |
| scan.max-retries            | 否       | 字符串    | 读取任务在失败时可以重试的最大次数。默认值：`1`。如果读取任务的重试次数超过此限制，读取任务将返回错误。 |

### Flink DataStream参数

以下参数仅适用于Flink DataStream读取方法。

| 参数    | 必填 | 数据类型 | 描述                                                  |
| ------------ | -------- | --------- | ------------------------------------------------------------ |
| scan.columns | 否       | 字符串    | 要读取的列。可以指定多列，这些列必须用逗号（,）分隔。 |
| scan.filter  | 否       | 字符串    | 筛选数据的筛选条件。 |


假设您在 Flink 中创建了一个由三列组成的表，分别是 `c1`、 `c2`、 `c3`。要读取这个 Flink 表中 `c1` 列的值等于 `100` 的行，您可以指定两个过滤条件："scan.columns, "c1"` 和 `"scan.filter, "c1 = 100"`。

## StarRocks 和 Flink 的数据类型映射关系

以下数据类型映射仅适用于 Flink 从 StarRocks 读取数据的情况。有关 Flink 将数据写入 StarRocks 所使用的数据类型映射，请参见[从 Apache Flink® 持续加载数据](../loading/Flink-connector-starrocks.md)。

| StarRocks  | Flink     |
| ---------- | --------- |
| NULL       | NULL      |
| BOOLEAN    | BOOLEAN   |
| TINYINT    | TINYINT   |
| SMALLINT   | SMALLINT  |
| INT        | INT       |
| BIGINT     | BIGINT    |
| LARGEINT   | STRING    |
| FLOAT      | FLOAT     |
| DOUBLE     | DOUBLE    |
| DATE       | DATE      |
| DATETIME   | TIMESTAMP |
| DECIMAL    | DECIMAL   |
| DECIMALV2  | DECIMAL   |
| DECIMAL32  | DECIMAL   |
| DECIMAL64  | DECIMAL   |
| DECIMAL128 | DECIMAL   |
| CHAR       | CHAR      |
| VARCHAR    | STRING    |

## 例子

以下示例假设您已在 StarRocks 集群中创建了一个名为 `test` 的数据库，并且您拥有 `root` 用户的权限。

> **注意**
>
> 如果读取任务失败，您必须重新创建它。

### 数据示例

1. 进入 `test` 数据库并创建一个名为 `score_board` 的表。

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
   21 rows in set (0.00 sec)
   ```

### 使用 Flink SQL 读取数据

1. 在您的 Flink 集群中，根据源 StarRocks 表（在本示例中为 `score_board`）的模式创建名为 `flink_test` 的表。在创建表的命令中，您必须配置读取任务属性，包括 Flink 连接器信息、源 StarRocks 数据库以及源 StarRocks 表的信息。

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

使用 Flink SQL 读取数据时，请注意以下几点：

- 您只能使用类似 `SELECT ... FROM <table_name> WHERE ...` 的 SQL 语句从 StarRocks 读取数据。在所有聚合函数中，仅支持 `count`。
- 支持谓词下推。例如，如果您的查询包含过滤条件 `char_1 <> 'A' and int_1 = -126`，则过滤条件将被下推到 Flink 连接器，并在查询运行之前转换为可以由 StarRocks 执行的语句。您无需执行额外的配置。
- 不支持 LIMIT 语句。
- StarRocks 不支持检查点机制。因此，如果读取任务失败，则无法保证数据的一致性。

### 使用 Flink DataStream 读取数据

1. 将以下依赖项添加到 `pom.xml` 文件中：

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

   请将上述代码示例中的 `x.x.x` 替换为您正在使用的最新 Flink 连接器版本。请参阅[版本信息](https://search.maven.org/search?q=g:com.starrocks)。

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
               env.execute("StarRocks flink source");
           }

       }
   ```

## 下一步怎么办

在 Flink 成功从 StarRocks 读取数据之后，您可以使用[Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)来监控读取任务。例如，您可以在**Metrics**页面上查看`totalScannedRows`指标，以获取成功读取的行数。您还可以使用 Flink SQL 对已读取的数据进行连接等计算。