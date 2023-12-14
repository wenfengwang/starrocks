---
displayed_sidebar: "Chinese"
---

# 使用 Flink 连接器从 StarRocks 读取数据

StarRocks 提供一种自行开发的名为 Apache Flink®（简称 Flink）的连接器，名为 StarRocks Connector 以帮助你使用 Flink 从 StarRocks 集群中批量读取数据。

Flink 连接器支持两种读取方法：Flink SQL 和 Flink DataStream。推荐使用 Flink SQL。

> **注意**
>
> Flink 连接器还支持将 Flink 读取的数据写入另一个 StarRocks 集群或存储系统。请参见 [从 Apache Flink® 连续加载数据](../loading/Flink-connector-starrocks.md)。

## 背景信息

与 Flink 提供的 JDBC 连接器不同，StarRocks 的 Flink 连接器支持并行从 StarRocks 集群的多个 BE 中读取数据，大大加速了读取任务。以下比较显示了两个连接器的实现差异。

- StarRocks 的 Flink 连接器

  使用 StarRocks 的 Flink 连接器，Flink 首先可以从负责的 FE 获取查询计划，然后将获取的查询计划作为参数分发给所有涉及的 BE，并最终获取 BE 返回的数据。

  ![- StarRocks 的 Flink 连接器](../assets/5.3.2-1.png)

- Flink 的 JDBC 连接器

  使用 Flink 的 JDBC 连接器，Flink 只能逐个从单个 FE 读取数据。数据读取速度较慢。

  ![Flink 的 JDBC 连接器](../assets/5.3.2-2.png)

## 版本要求

| 连接器 | Flink                   | StarRocks     | Java | Scala     |
|-------|-------------------------|---------------| ---- |-----------|
| 1.2.8 | 1.13,1.14,1.15,1.16,1.17 | 2.1 及更高版本| 8    | 2.11,2.12 |
| 1.2.7 | 1.11,1.12,1.13,1.14,1.15 | 2.1 及更高版本| 8    | 2.11,2.12 |

## 先决条件

已部署 Flink。如果尚未部署 Flink，请按照以下步骤进行部署：

1. 在操作系统中安装 Java 8 或 Java 11 以确保 Flink 可以正常运行。你可以使用以下命令检查你的 Java 安装版本：

   ```SQL
   java -version
   ```

   例如，如果返回以下信息，表示已安装 Java 8：

   ```SQL
   openjdk version "1.8.0_322"
   OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
   OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
   ```

2. 下载并解压缩你选择的 [Flink 软件包](https://flink.apache.org/downloads.html)。

   > **注意**
   >
   > 我们建议你使用 Flink v1.14 或更高版本。最低支持的 Flink 版本是 v1.11。

   ```SQL
   # 下载 Flink 软件包。
   wget https://dlcdn.apache.org/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
   # 解压缩 Flink 软件包。
   tar -xzf flink-1.14.5-bin-scala_2.11.tgz
   # 切换到 Flink 目录。
   cd flink-1.14.5
   ```

3. 启动你的 Flink 集群。

   ```SQL
   # 启动你的 Flink 集群。
   ./bin/start-cluster.sh
         
   # 显示以下信息时，表示你的 Flink 集群已成功启动：
   Starting cluster.
   Starting standalonesession daemon on host.
   Starting taskexecutor daemon on host.
   ```

你也可以按照[Flink文档](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)中提供的说明部署 Flink。

## 开始之前

按照以下步骤部署 Flink 连接器：

1. 选择并下载与你使用的 Flink 版本相匹配的 [flink-connector-starrocks](https://github.com/StarRocks/flink-connector-starrocks/releases) JAR 软件包。

   > **注意**
   >
   > 我们建议你下载版本为 1.2.x 或更高版本且与你使用的 Flink 版本的前两位数字相同的 Flink 连接器软件包。例如，如果你使用 Flink v1.14.x，你可以下载 `flink-connector-starrocks-1.2.4_flink-1.14_x.yy.jar`。

2. 如果需要进行代码调试，编译 Flink 连接器软件包以适应你的业务需求。

3. 将你下载或编译的 Flink 连接器软件包放入 Flink 的 `lib` 目录中。

4. 重新启动你的 Flink 集群。

## 参数

### 通用参数

以下参数适用于 Flink SQL 和 Flink DataStream 读取方法。

| 参数                        | 是否必需 | 数据类型 | 描述                                                          |
| --------------------------- | -------- | --------- | ------------------------------------------------------------ |
| connector                   | 是       | STRING    | 你要使用的用于读取数据的连接器类型。将值设置为 `starrocks`。                                |
| scan-url                    | 是       | STRING    | 用于从 Web 服务器连接到 FE 的地址。格式：`<fe_host>:<fe_http_port>`。默认端口为 `8030`。可以指定多个地址，必须使用逗号 (,) 分隔。例如：`192.168.xxx.xxx:8030,192.168.xxx.xxx:8030`。 |
| jdbc-url                    | 是       | STRING    | 用于连接 FE 的 MySQL 客户端的地址。格式：`jdbc:mysql://<fe_host>:<fe_query_port>`。默认端口号为 `9030`。 |
| username                    | 是       | STRING    | 你的 StarRocks 集群帐户的用户名。该帐户必须具有你要读取的 StarRocks 表的读取权限。见 [用户权限](../administration/User_privilege.md)。 |
| password                    | 是       | STRING    | 你的 StarRocks 集群帐户的密码。              |
| database-name               | 是       | STRING    | 你要读取的 StarRocks 表所属的 StarRocks 数据库的名称。 |
| table-name                  | 是       | STRING    | 你要读取的 StarRocks 表的名称。            |
| scan.connect.timeout-ms     | 否       | STRING    | Flink 连接器连接到你的 StarRocks 集群超时的最长时间。单位：毫秒。默认值：`1000`。如果建立连接所用的时间超过此限制，读取任务失败。 |
| scan.params.keep-alive-min  | 否       | STRING    | 读取任务保持存活的最长时间。存活时间是通过轮询机制定期检查的。单位：分钟。默认值：`10`。建议将此参数设置为大于或等于 `5` 的值。 |
| scan.params.query-timeout-s | 否       | STRING    | 读取任务超时的最长时间。任务执行期间检查超时持续时间。单位：秒。默认值：`600`。如果在时间持续时间结束后没有返回读取结果，读取任务将停止。 |
| scan.params.mem-limit-byte  | 否       | STRING    | 每个 BE 上每个查询允许的最大内存量。单位：字节。默认值：`1073741824`，相当于 1 GB。 |
| scan.max-retries            | 否       | STRING    | 可以重试读取任务的最大次数。默认值：`1`。如果重试读取任务的次数超过此限制，读取任务将返回错误。 |

### Flink DataStream 参数

以下参数仅适用于 Flink DataStream 读取方法。

| 参数          | 是否必需 | 数据类型 | 描述                                                          |
| ------------ | -------- | --------- | ------------------------------------------------------------ |
| scan.columns | 否       | STRING    | 你要读取的列。可以指定多个列，必须使用逗号 (,) 分隔。 |
| scan.filter  | 否       | STRING    | 你要基于其进行数据筛选的过滤条件。 |

假设你在 Flink 中创建了一个由三列 `c1`、`c2`、`c3` 组成的表，要读取这张 Flink 表中 `c1` 列的值等于 `100` 的行，你可以指定两个筛选条件 `"scan.columns, "c1"` 和 `"scan.filter, "c1 = 100"`。

## StarRocks 和 Flink 之间的数据类型映射

以下数据类型映射仅适用于Flink从StarRocks读取数据。有关用于Flink将数据写入StarRocks的数据类型映射，请参见[从Apache Flink®连续加载数据](../loading/Flink-connector-starrocks.md)。

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

## 示例

以下示例假定您已在StarRocks集群中创建名为`test`的数据库，并拥有用户`root`的权限。

> **注意**
>
> 如果读取任务失败，必须重新创建它。

### 数据示例

1. 进入`test`数据库并创建名为`score_board`的表。

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

2. 向`score_board`表中插入数据。

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

3. 查询`score_board`表。

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

### 使用Flink SQL读取数据

1. 在Flink集群中，根据源StarRocks表的架构（本示例中为`score_board`）创建名为`flink_test`的表。在表创建命令中，必须配置读取任务属性，包括有关Flink连接器、源StarRock数据库和源StarRocks表的信息。

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

2. 使用SELECT从StarRocks读取数据。

   ```SQL
   SELECT id, name FROM flink_test WHERE score > 20;
   ```

在使用Flink SQL读取数据时，请注意以下几点：

- 您只能使用类似`SELECT ... FROM <table_name> WHERE ...`的SQL语句从StarRocks读取数据。在所有聚合函数中，只支持`count`。
- 支持谓词下推。例如，如果您的查询包含筛选条件`char_1 <> 'A' and int_1 = -126`，则筛选条件将被下推到Flink连接器并转换为StarRocks可以执行的语句，而无需执行额外的配置。
- 不支持LIMIT语句。
- StarRocks不支持检查点机制。因此，如果读取任务失败，无法保证数据一致性。

### 使用Flink DataStream读取数据

1. 将以下依赖项添加到`pom.xml`文件中：

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

   您必须使用实际使用的最新Flink连接器版本替换上述示例代码中的`x.x.x`。参见[版本信息](https://search.maven.org/search?q=g:com.starrocks)。

2. 调用Flink连接器从StarRocks读取数据：

   ```JAVA
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

## 接下来怎么办

在 Flink 从 StarRocks 成功读取数据后，您可以使用 [Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui) 监控读取任务。例如，您可以在 WebUI 的 **Metrics** 页面查看 `totalScannedRows` 指标，以获取成功读取的行数。您也可以使用 Flink SQL 对所读取的数据进行联接等计算操作。