---
displayed_sidebar: English
---

# 从 MySQL 实时同步

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote

StarRocks 能够在数秒内支持从 MySQL 到 StarRocks 的实时数据同步，提供超低延迟的大规模实时分析能力，使用户能够查询实时发生的数据。

本教程将帮助您学习如何为您的企业和用户提供实时分析。它演示了如何使用 StarRocks 迁移工具（SMT）、Flink、Flink CDC Connector 和 flink-starrocks-connector 等工具，实现从 MySQL 到 StarRocks 的实时数据同步。

<InsertPrivNote />


## 工作原理

下图展示了整个同步过程。

![img](../assets/4.9.2.png)

从 MySQL 到 StarRocks 的实时同步分为两个阶段：同步数据库与表结构和同步数据。首先，SMT 将 MySQL 数据库与表结构转换成 StarRocks 的建表语句。然后，Flink 集群运行 Flink 作业，将 MySQL 的全量和增量数据同步到 StarRocks。

> **注意**
> 同步过程保证了*精确一次*的语义。

**同步流程**：

1. 1. 同步数据库与表结构。

   SMT 读取待同步的 MySQL 数据库与表的结构，并生成 SQL 文件，用于在 StarRocks 中创建目标数据库与表。此操作基于 SMT 配置文件中的 MySQL 和 StarRocks 信息。

2. 2. 同步数据。

      a. Flink SQL 客户端执行数据加载语句 INSERT INTO SELECT，向 Flink 集群提交一个或多个 Flink 作业。

   b. Flink集群运行Flink作业获取数据。[Flink CDC Connector](https://github.com/ververica/flink-cdc-connectors/blob/master/docs/content/quickstart/build-real-time-data-lake-tutorial.md)首先从源数据库读取完整的历史数据，然后无缝切换到增量读取，并将数据发送到flink-starrocks-connector。

      c. flink-starrocks-connector 以小批量方式积累数据，并将每批数据同步到 StarRocks。

> **注意**
> 只有 MySQL 中的**数据操纵语言（DML）**操作可以同步到 StarRocks。**数据定义语言（DDL）**操作无法同步。

## 应用场景

从 MySQL 到 StarRocks 的实时同步适用于数据不断变化的广泛场景。以“商品销量实时排名”这一实际用例为例：

Flink 根据 MySQL 中的原始订单表计算商品销量的实时排名，并实时将排名同步到 StarRocks 的主键表中。用户可以将可视化工具连接到 StarRocks，实时查看排名，以获得即时的运营洞察。

## 准备工作

### 下载并安装同步工具

要从 MySQL 同步数据，您需要安装以下工具：SMT、Flink、Flink CDC Connector 和 flink-starrocks-connector。

1. 下载并安装 Flink，并启动 Flink 集群。您也可以按照[Flink官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)的指引进行操作。

      a. 在运行 Flink 之前，请在您的操作系统中安装 Java 8 或 Java 11。您可以运行以下命令检查已安装的 Java 版本。

   ```Bash
       # View the Java version.
       java -version
   
       # Java 8 is installed if the following output is returned.
       java version "1.8.0_301"
       Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
       Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
   ```

   b. 下载[Flink安装包](https://flink.apache.org/downloads.html)并解压。我们推荐使用Flink 1.14或更新版本。允许的最低版本为Flink 1.11。本教程使用Flink 1.14.5。

   ```Bash
      # Download Flink.
      wget https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
      # Decompress Flink.  
      tar -xzf flink-1.14.5-bin-scala_2.11.tgz
      # Go to the Flink directory.
      cd flink-1.14.5
   ```

      c. 启动 Flink 集群。

   ```Bash
      # Start the Flink cluster.
      ./bin/start-cluster.sh
   
      # The Flink cluster is started if the following output is returned.
      Starting cluster.
      Starting standalonesession daemon on host.
      Starting taskexecutor daemon on host.
   ```

2. 下载 [Flink CDC connector](https://github.com/ververica/flink-cdc-connectors/releases)。本教程使用 MySQL 作为数据源，因此下载 `flink-sql-connector-mysql-cdc-x.x.x.jar`。连接器版本必须与 [Flink](https://github.com/ververica/flink-cdc-connectors/releases) 版本相匹配。详细版本映射请参见[Supported Flink Versions](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions)。本教程使用 Flink 1.14.5，您可以下载 `flink-sql-connector-mysql-cdc-2.2.0.jar`。

   ```Bash
   wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.1.1/flink-sql-connector-mysql-cdc-2.2.0.jar
   ```

3. 下载 [flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks)。版本必须与 Flink 版本相匹配。

      > flink-connector-starrocks 包 x.x.x_flink-y.yy_z.zz.jar 包含三个版本号：
   -    x.x.x 是 flink-connector-starrocks 的版本号。
   -    y.yy 是支持的 Flink 版本。
   -    z.zz 是 Flink 支持的 Scala 版本。如果 Flink 版本为 1.14.x 或更早版本，您必须下载对应 Scala 版本的包。
      > 本教程使用 Flink 1.14.5 和 Scala 2.11。因此，您可以下载以下包：1.2.3_flink-1.14_2.11.jar。

4. 将 Flink CDC Connector（flink-sql-connector-mysql-cdc-2.2.0.jar）和 flink-connector-starrocks（1.2.3_flink-1.14_2.11.jar）的 JAR 包移至 Flink 的 lib 目录下。

      > **注意**
      > 如果一个Flink集群已经在您的系统中运行，您必须停止Flink集群并重新启动它以加载和验证JAR包。
   ```Bash
   $ ./bin/stop-cluster.sh
   $ ./bin/start-cluster.sh
   ```

5. 下载并解压[SMT 包](https://www.starrocks.io/download/community)，将其放置在`flink-1.14.5`目录中。StarRocks为Linux x86和macOS ARM64提供了SMT包。您可以根据您的操作系统和CPU选择合适的版本。

   ```Bash
   # for Linux x86
   wget https://releases.starrocks.io/resources/smt.tar.gz
   # for macOS ARM64
   wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
   ```

### 启用 MySQL 二进制日志

为了实时同步 MySQL 数据，系统需要从 MySQL 二进制日志（binlog）读取数据，解析数据，然后将数据同步到 StarRocks。请确保 MySQL 二进制日志已启用。

1. 编辑 MySQL 配置文件 my.cnf（默认路径：/etc/my.cnf），以启用 MySQL 二进制日志。

   ```Bash
   # Enable MySQL Binlog.
   log_bin = ON
   # Configure the save path for the Binlog.
   log_bin =/var/lib/mysql/mysql-bin
   # Configure server_id.
   # If server_id is not configured for MySQL 5.7.3 or later, the MySQL service cannot be used. 
   server_id = 1
   # Set the Binlog format to ROW. 
   binlog_format = ROW
   # The base name of the Binlog file. An identifier is appended to identify each Binlog file.
   log_bin_basename =/var/lib/mysql/mysql-bin
   # The index file of Binlog files, which manages the directory of all Binlog files.  
   log_bin_index =/var/lib/mysql/mysql-bin.index
   ```

2. 执行以下命令重启 MySQL，使修改后的配置生效。

   ```Bash
   # Use service to restart MySQL.
   service mysqld restart
   # Use mysqld script to restart MySQL.
   /etc/init.d/mysqld restart
   ```

3. 连接到 MySQL 并检查是否启用了二进制日志。

   ```Plain
   -- Connect to MySQL.
   mysql -h xxx.xx.xxx.xx -P 3306 -u root -pxxxxxx
   
   -- Check whether MySQL binary log is enabled.
   mysql> SHOW VARIABLES LIKE 'log_bin'; 
   +---------------+-------+
   | Variable_name | Value |
   +---------------+-------+
   | log_bin       | ON    |
   +---------------+-------+
   1 row in set (0.00 sec)
   ```

## 同步数据库与表结构

1. 编辑 SMT 配置文件。进入 SMT conf 目录，编辑配置文件 config_prod.conf，填写 MySQL 连接信息、待同步的数据库与表的匹配规则、flink-starrocks-connector 的配置信息等。

   ```Bash
   [db]
   host = xxx.xx.xxx.xx
   port = 3306
   user = user1
   password = xxxxxx
   
   [other]
   # Number of BEs in StarRocks
   be_num = 3
   # `decimal_v3` is supported since StarRocks-1.18.1.
   use_decimal_v3 = true
   # File to save the converted DDL SQL
   output_dir = ./result
   
   [table-rule.1]
   # Pattern to match databases for setting properties
   database = ^demo.*$
   # Pattern to match tables for setting properties
   table = ^.*$
   
   ############################################
   ### Flink sink configurations
   ### DO NOT set `connector`, `table-name`, `database-name`. They are auto-generated.
   ############################################
   flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
   flink.starrocks.load-url= <fe_host>:<fe_http_port>
   flink.starrocks.username=user2
   flink.starrocks.password=xxxxxx
   flink.starrocks.sink.properties.format=csv
   flink.starrocks.sink.properties.column_separator=\x01
   flink.starrocks.sink.properties.row_delimiter=\x02
   flink.starrocks.sink.buffer-flush.interval-ms=15000
   ```

-     [db]：MySQL 连接信息。
      -     host：MySQL 服务器的 IP 地址。
      -     port：MySQL 数据库的端口号，默认为 3306。
      -     user：访问 MySQL 数据库的用户名。
      -     password：用户名的密码。

-     [table-rule]：数据库与表的匹配规则及对应的 flink-connector-starrocks 配置。

      -     Database, table：MySQL 中的数据库和表的名称。支持正则表达式。
      -  `flink.starrocks.*`：flink-connector-starrocks 的配置信息。更多配置和信息，请参见 [flink-connector-starrocks](../loading/Flink-connector-starrocks.md) 文档。

            > 如果需要为不同的表使用不同的 flink-connector-starrocks 配置，例如某些表频繁更新需要加速数据加载，参见[Use different flink-connector-starrocks configurations for different tables](#use-different-flink-connector-starrocks-configurations-for-different-tables)。如果需要将 MySQL 分片后的多个表同步到 StarRocks 的同一张表，请参见[Synchronize multiple tables after MySQL sharding to one table in StarRocks](#synchronize-multiple-tables-after-mysql-sharding-to-one-table-in-starrocks)。

-     [other]：其他信息
      -     be_num：StarRocks 集群中 BE 的数量（此参数将用于后续 StarRocks 建表时设置合理的分区数量）。
      -  `use_decimal_v3`: 是否启用[Decimal V3](../sql-reference/sql-statements/data-types/DECIMAL.md)。启用 Decimal V3 后，MySQL 十进制数据在同步到 StarRocks 时将转换为 Decimal V3 数据。
      -     output_dir：保存将要生成的 SQL 文件的路径。SQL 文件将用于在 StarRocks 中创建数据库与表，以及提交 Flink 作业到 Flink 集群。默认路径为 ./result，建议保留默认设置。

2. 运行 SMT 读取 MySQL 中的数据库与表结构，并根据配置文件在 ./result 目录生成 SQL 文件。starrocks-create.all.sql 文件用于在 StarRocks 中创建数据库与表，flink-create.all.sql 文件用于提交 Flink 作业到 Flink 集群。

   ```Bash
   # Run the SMT.
   ./starrocks-migrate-tool
   
   # Go to the result directory and check the files in this directory.
   cd result
   ls result
   flink-create.1.sql    smt.tar.gz              starrocks-create.all.sql
   flink-create.all.sql  starrocks-create.1.sql
   ```

3. 执行以下命令连接到 StarRocks，并执行 `starrocks-create.all.sql` 文件以在 StarRocks 中创建数据库和表。我们推荐使用 SQL 文件中的默认建表语句来创建表的[主键表](../table_design/table_types/primary_key_table.md)。

      > **注意**
      > 您也可以根据业务需求修改建表语句，创建非主键表。但请注意，源 MySQL 数据库中的 DELETE 操作无法同步到非主键表。创建此类表时请谨慎。

   ```Bash
   mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
   ```

   如果数据在写入目标 StarRocks 表之前需要经过 Flink 处理，则源表和目标表之间的表结构可能会不同。在这种情况下，您必须修改建表语句。例如，在本例中，目标表只需要 product_id 和 product_name 列以及商品销量的实时排名。您可以使用以下建表语句。

   ```Bash
   CREATE DATABASE IF NOT EXISTS `demo`;
   
   CREATE TABLE IF NOT EXISTS `demo`.`orders` (
   `product_id` INT(11) NOT NULL COMMENT "",
   `product_name` STRING NOT NULL COMMENT "",
   `sales_cnt` BIGINT NOT NULL COMMENT ""
   ) ENGINE=olap
   PRIMARY KEY(`product_id`)
   DISTRIBUTED BY HASH(`product_id`)
   PROPERTIES (
   "replication_num" = "3"
   );
   ```

      > **注意**
      > 从 v2.5.7 版本开始，StarRocks 可以在创建表或添加分区时自动设置桶（BUCKETS）的数量。您不再需要手动设置桶的数量。有关详细信息，请参见[determine the number of buckets](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

## 同步数据

运行 Flink 集群并提交 Flink 作业，以持续同步 MySQL 的全量和增量数据到 StarRocks。

1. 进入 Flink 目录，执行以下命令在 Flink SQL 客户端运行 flink-create.all.sql 文件。

   ```Bash
   ./bin/sql-client.sh -f flink-create.all.sql
   ```

   该 SQL 文件定义了源表和接收表的动态表、查询语句 INSERT INTO SELECT，并指定了连接器、源数据库和目标数据库。执行该文件后，会向 Flink 集群提交 Flink 作业，开始数据同步。

      > **注意**
   - 确保 Flink 集群已启动。您可以通过运行 flink/bin/start-cluster.sh 启动 Flink 集群。
   - 如果您的 Flink 版本低于 1.13，您可能无法直接运行 SQL 文件 flink-create.all.sql。您需要在 SQL 客户端的命令行界面（CLI）中逐条执行该文件中的 SQL 语句，并且需要对 \ 字符进行转义。

   ```Bash
   'sink.properties.column_separator' = '\\x01'
   'sink.properties.row_delimiter' = '\\x02'  
   ```

   **同步过程中的数据处理**：

   如果需要在同步过程中对数据进行处理，例如执行 GROUP BY 或 JOIN 操作，您可以修改 flink-create.all.sql 文件。以下例子通过执行 COUNT(*) 和 GROUP BY 来计算商品销量的实时排名。

   ```Bash
       $ ./bin/sql-client.sh -f flink-create.all.sql
       No default environment is specified.
       Searching for '/home/disk1/flink-1.13.6/conf/sql-client-defaults.yaml'...not found.
       [INFO] Executing SQL from file.
   
       Flink SQL> CREATE DATABASE IF NOT EXISTS `default_catalog`.`demo`;
       [INFO] Execute statement succeed.
   
       -- Create a dynamic table `source table` based on the order table in MySQL.
       Flink SQL> 
       CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_src` (`order_id` BIGINT NOT NULL,
       `product_id` INT NULL,
       `order_date` TIMESTAMP NOT NULL,
       `customer_name` STRING NOT NULL,
       `product_name` STRING NOT NULL,
       `price` DECIMAL(10, 5) NULL,
       PRIMARY KEY(`order_id`)
       NOT ENFORCED
       ) with ('connector' = 'mysql-cdc',
       'hostname' = 'xxx.xx.xxx.xxx',
       'port' = '3306',
       'username' = 'root',
       'password' = '',
       'database-name' = 'demo',
       'table-name' = 'orders'
       );
       [INFO] Execute statement succeed.
   
       -- Create a dynamic table `sink table`.
       Flink SQL> 
       CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_sink` (`product_id` INT NOT NULL,
       `product_name` STRING NOT NULL,
       `sales_cnt` BIGINT NOT NULL,
       PRIMARY KEY(`product_id`)
       NOT ENFORCED
       ) with ('sink.max-retries' = '10',
       'jdbc-url' = 'jdbc:mysql://<fe_host>:<fe_query_port>',
       'password' = '',
       'sink.properties.strip_outer_array' = 'true',
       'sink.properties.format' = 'json',
       'load-url' = '<fe_host>:<fe_http_port>',
       'username' = 'root',
       'sink.buffer-flush.interval-ms' = '15000',
       'connector' = 'starrocks',
       'database-name' = 'demo',
       'table-name' = 'orders'
       );
       [INFO] Execute statement succeed.
   
       -- Implement real-time ranking of commodity sales, where `sink table` is dynamically updated to reflect data changes in `source table`.
       Flink SQL> 
       INSERT INTO `default_catalog`.`demo`.`orders_sink` select product_id,product_name, count(*) as cnt from `default_catalog`.`demo`.`orders_src` group by product_id,product_name;
       [INFO] Submitting SQL update statement to the cluster...
       [INFO] SQL update statement has been successfully submitted to the cluster:
       Job ID: 5ae005c4b3425d8bb13fe660260a35da
   ```

   如果您只需要同步部分数据，例如支付时间晚于 2021 年 12 月 21 日的数据，您可以在 INSERT INTO SELECT 的 WHERE 子句中设置过滤条件，例如 WHERE pay_dt > '2021-12-21'。不满足此条件的数据将不会同步到 StarRocks。

   如果返回以下结果，则表明 Flink 作业已提交，进行全量和增量同步。

   ```SQL
   [INFO] Submitting SQL update statement to the cluster...
   [INFO] SQL update statement has been successfully submitted to the cluster:
   Job ID: 5ae005c4b3425d8bb13fe660260a35da
   ```

2. 您可以使用[Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)或运行`bin/flink list -running`命令在您的Flink SQL客户端上查看正在运行的Flink作业和作业ID。

-     Flink WebUI
![图片](../assets/4.9.3.png)

-     bin/flink list -running

   ```Bash
       $ bin/flink list -running
       Waiting for response...
       ------------------ Running/Restarting Jobs -------------------
       13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
       --------------------------------------------------------------
   ```

      > **注意**
      > 如果作业异常，可以通过 Flink WebUI或查看`/log`目录下的日志文件进行故障排除。

## 常见问题解答

### 为不同的表使用不同的 flink-connector-starrocks 配置

如果数据源中的某些表更新频繁，并且您希望加快 flink-connector-starrocks 的加载速度，则必须在 SMT 配置文件 config_prod.conf 中为每个表设置单独的 flink-connector-starrocks 配置。

```Bash
[table-rule.1]
# Pattern to match databases for setting properties
database = ^order.*$
# Pattern to match tables for setting properties
table = ^.*$

############################################
### Flink sink configurations
### DO NOT set `connector`, `table-name`, `database-name`. They are auto-generated
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url= <fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=15000[table-rule.2]
# Pattern to match databases for setting properties
database = ^order2.*$
# Pattern to match tables for setting properties
table = ^.*$

############################################
### Flink sink configurations
### DO NOT set `connector`, `table-name`, `database-name`. They are auto-generated
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url= <fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=10000
```

### 将 MySQL 分片后的多个表同步到 StarRocks 的同一张表

在进行分片后，一个 MySQL 表中的数据可能被拆分到多个表，甚至分布到多个数据库中。所有这些表都具有相同的结构。在这种情况下，您可以设置 [table-rule] 来将这些表同步到 StarRocks 的同一张表中。例如，如果 MySQL 有两个数据库 edu_db_1 和 edu_db_2，每个数据库都有两个表 course_1 和 course_2，并且所有表的结构相同。您可以使用以下 [table-rule] 配置将所有表同步到 StarRocks 的一张表中。

> **注意**
> StarRocks 表的默认名称为 `course__auto_shard`。如果您需要使用不同的名称，可以在 SQL 文件 `starrocks-create.all.sql` 和 `flink-create.all.sql` 中修改它。

```Bash
[table-rule.1]
# Pattern to match databases for setting properties
database = ^edu_db_[0-9]*$
# Pattern to match tables for setting properties
table = ^course_[0-9]*$

############################################
### Flink sink configurations
### DO NOT set `connector`, `table-name`, `database-name`. They are auto-generated
############################################
flink.starrocks.jdbc-url = jdbc: mysql://xxx.xxx.x.x:xxxx
flink.starrocks.load-url = xxx.xxx.x.x:xxxx
flink.starrocks.username = user2
flink.starrocks.password = xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
flink.starrocks.sink.buffer-flush.interval-ms = 5000
```

### 以 JSON 格式导入数据

在上面的例子中，数据是以 CSV 格式导入的。如果您无法选择合适的分隔符，您需要替换 [table-rule] 中的 flink.starrocks.* 参数。

```Plain
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
```

传入以下参数后，数据将以 JSON 格式导入。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注意**
> 这种方法**可能会**稍微**降低**加载速度。

### 将多条 INSERT INTO 语句作为一个 Flink 作业执行

您可以使用[STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements)语法在`flink-create.all.sql`文件中执行多个INSERT INTO语句，作为一个Flink作业，这样可以避免多个语句占用过多的Flink作业资源，并提高执行多个查询的效率。

> **注意**
> Flink 从 1.13 版本开始支持 **STATEMENT SET** 语法。

1. 打开 result/flink-create.all.sql 文件。

2. 修改文件中的 SQL 语句。将所有 INSERT INTO 语句移动到文件末尾，在第一个 INSERT INTO 语句之前添加 EXECUTE STATEMENT SET BEGIN，最后一个 INSERT INTO 语句之后添加 END;。

> **注意**
> CREATE DATABASE 和 CREATE TABLE 语句的位置保持不变。

```SQL
CREATE DATABASE IF NOT EXISTS db;
CREATE TABLE IF NOT EXISTS db.a1;
CREATE TABLE IF NOT EXISTS db.b1;
CREATE TABLE IF NOT EXISTS db.a2;
CREATE TABLE IF NOT EXISTS db.b2;
EXECUTE STATEMENT SET 
BEGIN-- one or more INSERT INTO statements
INSERT INTO db.a1 SELECT * FROM db.b1;
INSERT INTO db.a2 SELECT * FROM db.b2;
END;
```
