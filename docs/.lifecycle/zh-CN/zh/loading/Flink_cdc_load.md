```yaml
---
displayed_sidebar: "Chinese"
---

# 实时同步 MySQL 数据至 StarRocks

This article describes how to synchronize data from MySQL to StarRocks in real time (sub-second), to support the real-time analysis and processing needs of enterprises.

> **Note**
>
> The import operation requires INSERT permission for the target table. If your user account does not have INSERT permission, please refer to [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) to grant permissions to the user.

## Basic Principle

![MySQL synchronization](../assets/4.9.2.png)

The real-time synchronization of MySQL to StarRocks is divided into two stages: synchronization of the library table structure and synchronization of data. First, the StarRocks Migration Tool (data migration tool, hereinafter referred to as SMT) translates the library table structure of MySQL into the build and create table statements of StarRocks based on its configuration file. Then, the Flink cluster runs the Flink job to synchronize the full and incremental data from MySQL to StarRocks. The specific synchronization process is as follows:

> Note:
> Real-time synchronization of MySQL to StarRocks ensures end-to-end exactly-once semantic consistency.

1. **Synchronize the library table structure**

   According to the source MySQL and target StarRocks information in its configuration file, SMT reads the library table structure to be synchronized from MySQL and generates an SQL file for the creation of the corresponding target library table in StarRocks.

2. **Synchronize data**

   The Flink SQL client executes the SQL statement to import data (`INSERT INTO SELECT` statement) and submits one or more long-running Flink jobs to the Flink cluster. The Flink cluster runs the Flink job, the [Flink cdc connector](https://ververica.github.io/flink-cdc-connectors/master/content/Quickstart/build-real-time-data-lake-tutorial.html) first reads the historical full data of the database, then seamlessly switches to incremental reading, and sends it to the flink-starrocks-connector. Finally, the flink-starrocks-connector aggregates micro-batch data for synchronization to StarRocks.

   > **Note**
   >
   > Only DML synchronization is supported, not DDL synchronization.

## Business Scenario

Take the real-time ranking of accumulated sales volume of goods as an example. The original order table stored in MySQL is processed by Flink to calculate the real-time ranking of product sales volume, and is synchronized to the primary key model table in StarRocks in real time. Finally, users can connect to StarRocks through visualization tools to view the real-time updated ranking.

## Preparation

### Download and Install Synchronization Tools

Synchronization requires the use of SMT, Flink, Flink CDC connector, and flink-starrocks-connector. The download and installation steps are as follows:

1. **Download, install, and start the Flink cluster**.
   > Note: The download and installation methods can also refer to the [Flink official documentation](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/).

   1. You need to install Java 8 or Java 11 in the operating system in advance to run Flink normally. You can check the installed Java version by the following command.

      ```Bash
      # View java version
      java -version
      
      # The following displays that Java 8 is installed
      openjdk version "1.8.0_322"
      OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
      OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
      ```

   2. Download and unzip [Flink](https://flink.apache.org/downloads.html). This example uses Flink 1.14.5.
      > Note: It is recommended to use version 1.14 and above, with a minimum support version of 1.11.

      ```Bash
      # Download Flink
      wget https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
      # Unzip Flink  
      tar -xzf flink-1.14.5-bin-scala_2.11.tgz
      # Enter the Flink directory
      cd flink-1.14.5
      ```

   3. Start the Flink cluster.

      ```Bash
      # Start the Flink cluster
      ./bin/start-cluster.sh
      
      # Returns the following message, indicating that the Flink cluster has been successfully started
      Starting cluster.
      Starting standalonesession daemon on host.
      Starting taskexecutor daemon on host.
      ```

2. **Download [Flink CDC connector](https://github.com/ververica/flink-cdc-connectors/releases)**. The data source in this example is MySQL, so download flink-sql-connector-**mysql**-cdc-x.x.x.jar. The version needs to support the corresponding Flink version. For the compatibility of the two versions, please refer to [Supported Flink Versions](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions). Since this article uses Flink 1.14.5, you can use flink-sql-connector-mysql-cdc-2.2.0.jar.

      ```Bash
      wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.2.0/flink-sql-connector-mysql-cdc-2.2.0.jar
      ```

3. **Download [flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks)**, and its version needs to correspond to the Flink version.

   > The JAR package of flink-connector-starrocks (**x.x.x_flink-y.yy_z.zz.jar**) will contain three version numbers:
   >
   > - The first version number x.x.x is the version number of flink-connector-starrocks.
   >
   > - The second version number y.yy is the version number of Flink it supports.
   >
   > - The third version number z.zz is the version number of Scala supported by Flink. If Flink is 1.14.x and earlier versions, you need to download flink-connector-starrocks with the Scala version number.

   Since this article uses Flink version 1.14.5 and Scala version 2.11, you can download the flink-connector-starrocks JAR package **1.2.3_flink-1.14_2.11.jar**.

4. Move the JAR packages of Flink CDC connector and flink-connector-starrocks **flink-sql-connector-mysql-cdc-2.2.0.jar** and **1.2.3_flink-1.14_2.11.jar** to the **lib** directory of Flink.

   > **Note**
   >
   > If Flink is already running, you need to stop Flink first, and then restart Flink to load and take effect the JAR packages.
   >
   > ```Bash
   > ./bin/stop-cluster.sh
   > ./bin/start-cluster.sh
   > ```

5. Download and unzip [SMT](https://www.mirrorship.cn/zh-CN/download/community), and place it under the **flink-1.14.5** directory. You can choose the corresponding SMT installation package based on the operating system and CPU architecture.

   ```Bash
   # For Linux x86
   wget https://releases.starrocks.io/resources/smt.tar.gz
   # For macOS ARM64
   wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
   ```

### Enable MySQL Binlog Log

You need to ensure that the MySQL Binlog log is enabled. When real-time synchronization is required, the MySQL Binlog log data needs to be read, parsed, and synchronized to StarRocks.

1. Edit the MySQL configuration file **my.cnf** (default path is **/etc/my.cnf**) to enable MySQL Binlog.
```
```Bash
   # 开启 Binlog 日志
   log_bin = ON
   # 设置 Binlog 的存储位置
   log_bin =/var/lib/mysql/mysql-bin
   # 设置 server_id 
   # 在 MySQL 5.7.3 及以后版本，如果没有 server_id，那么设置 binlog 后无法开启 MySQL 服务 
   server_id = 1
   # 设置 Binlog 模式为 ROW
   binlog_format = ROW
   # binlog 日志的基本文件名，后面会追加标识来表示每一个 Binlog 文件
   log_bin_basename =/var/lib/mysql/mysql-bin
   # binlog 文件的索引文件，管理所有 Binlog 文件的目录
   log_bin_index =/var/lib/mysql/mysql-bin.index
   ```

2. 执行如下命令，重启 MySQL，生效修改后的配置文件：

   ```Bash
    # 使用 service 启动
    service mysqld restart
    # 使用 mysqld 脚本启动
    /etc/init.d/mysqld restart
   ```

3. 连接 MySQL，执行如下语句确认是否已经开启 Binlog：

   ```Plain
   -- 连接 MySQL
   mysql -h xxx.xx.xxx.xx -P 3306 -u root -pxxxxxx

   -- 检查是否已经开启 MySQL Binlog，`ON`就表示已开启
   mysql> SHOW VARIABLES LIKE 'log_bin'; 
   +---------------+-------+
   | Variable_name | Value |
   +---------------+-------+
   | log_bin       | ON    |
   +---------------+-------+
   1 row in set (0.00 sec)
   ```

## 同步库表结构

1. 配置 SMT 配置文件。
   进入 SMT 的 **conf** 目录，编辑配置文件 **config_prod.conf**。例如源 MySQL 连接信息、待同步库表的匹配规则，flink-starrocks-connector 配置信息等。

    ```Bash
    [db]
    host = xxx.xx.xxx.xx
    port = 3306
    user = user1
    password = xxxxxx

    [other]
    # number of backends in StarRocks
    be_num = 3
    # `decimal_v3` is supported since StarRocks-1.18.1
    use_decimal_v3 = true
    # file to save the converted DDL SQL
    output_dir = ./result

    [table-rule.1]
    # pattern to match databases for setting properties
    database = ^demo.*$
    # pattern to match tables for setting properties
    table = ^.*$

    ############################################
    ### flink sink configurations
    ### DO NOT set `connector`, `table-name`, `database-name`, they are auto-generated
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

    - `[db]`：MySQL 的连接信息。
        - `host` ：MySQL 所在服务器的 IP 地址。
        - `port`：MySQL 端口号，默认为`3306`。
        - `user` ：用户名。
        - `password`：用户登录密码。

    - `[table-rule]` ：库表匹配规则，以及对应的flink-connector-starrocks 配置。

        > - 如果需要为不同表匹配不同的 flink-connector-starrocks 配置，例如部分表更新频繁，需要提高导入速度，请参见[补充说明](./Flink_cdc_load.md#常见问题)。
        > - 如果需要将 MySQL 分库分表后的多张表导入至 StarRocks的一张表中，请参见[补充说明](./Flink_cdc_load.md#常见问题)。

        - `database`、`table`：MySQL 中同步对象的库表名，支持正则表达式。

        - `flink.starrocks.*` ：flink-connector-starrocks 的配置信息，更多配置和说明，请参见 [Flink-connector-starrocks](./Flink-connector-starrocks.md#参数说明)。

    - `[other]` ：其他信息
        - `be_num`： StarRocks 集群的 BE 节点数（后续生成的 StarRocks 建表 SQL 文件会参考该参数，设置合理的分桶数量）。
        - `use_decimal_v3`：是否开启 [decimalV3](../sql-reference/sql-statements/data-types/DECIMAL.md)。开启后，MySQL 小数类型的数据同步至 StarRocks 时会转换为 decimalV3。
        - `output_dir` ：待生成的 SQL 文件的路径。SQL 文件会用于在 StarRocks 集群创建库表， 向 Flink 集群提交 Flink job。默认为 `./result`，不建议修改。

2. 执行如下命令，SMT 会读取 MySQL 中同步对象的库表结构，并且结合配置文件信息，在 **result** 目录生成 SQL 文件，用于  StarRocks 集群创建库表（**starrocks-create.all.sql**）， 用于向 Flink 集群提交同步数据的 flink job（**flink-create.all.sql**）。
   并且源表不同，则 **starrocks-create.all.sql** 中建表语句默认创建的数据模型不同。

   - 如果源表没有 Primary Key、 Unique Key，则默认创建明细模型。
   - 如果源表有 Primary Key、 Unique Key，则区分以下几种情况：
      - 源表是 Hive 表、ClickHouse MergeTree 表，则默认创建明细模型。
      - 源表是 ClickHouse SummingMergeTree表，则默认创建聚合模型。
      - 源表为其他类型，则默认创建主键模型。

    ```Bash
    # 运行 SMT
    ./starrocks-migrate-tool

    # 进入并查看 result 目录中的文件
    cd result
    ls result
    flink-create.1.sql    smt.tar.gz              starrocks-create.all.sql
    flink-create.all.sql  starrocks-create.1.sql
    ```

3. 执行如下命令，连接 StarRocks，并执行 SQL 文件 **starrocks-create.all.sql**，用于创建目标库和表。推荐使用 SQL 文件中默认的建表语句，本示例中建表语句默认创建的数据模型为[主键模型](../table_design/table_types/primary_key_table.md)。

    > **注意**
    >
    > - 您也可以根据业务需要，修改 SQL 文件中的建表语句，基于其他模型创建目标表。
    > - 如果您选择基于非主键模型创建目标表，StarRocks 不支持将源表中 DELETE 操作同步至非主键模型的表，请谨慎使用。

    ```Bash
    mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
    ```

    如果数据需要经过 Flink 处理后写入目标表，目标表与源表的结构不一样，则您需要修改 SQL 文件 **starrocks-create.all.sql** 中的建表语句。本示例中目标表仅需要保留商品 ID (product_id)、商品名称(product_name)，并且对商品销量进行实时排名，因此可以使用如下建表语句。

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
   >
   > 自 2.5.7 版本起，StarRocks 支持在建表和新增分区时自动设置分桶数量 (BUCKETS)，您无需手动设置分桶数量。更多信息，请参见 [确定分桶数量](../table_design/Data_distribution.md#确定分桶数量)。

## 同步数据

运行 Flink 集群，提交 Flink job，启动流式作业，源源不断将 MySQL 数据库中的全量和增量数据同步到 StarRocks 中。

1. 进入 Flink 目录，执行如下命令，在 Flink SQL 客户端运行 SQL 文件 **flink-create.all.sql**。

    该 SQL 文件定义了动态表 source table、sink table，查询语句 INSERT INTO SELECT，并且指定 connector、源数据库和目标数据库。Flink SQL 客户端执行该 SQL 文件后，向 Flink 集群提交一个 Flink job，开启同步任务。

    ```Bash
    ./bin/sql-client.sh -f flink-create.all.sql
    ```

    > **注意**
    >
    > - 需要确保 Flink 集群已经启动。可通过命令 `flink/bin/start-cluster.sh` 启动。
    >
    > - 如果您使用 Flink 1.13 之前的版本，则可能无法直接运行 SQL 文件 **flink-create.all.sql**。您需要在 SQL 客户端命令行界面，逐条执行 SQL 文件 **flink-create.all.sql** 中的 SQL 语句，并且需要做对`\`字符进行转义。
    >
    > ```Bash
    > 'sink.properties.column_separator' = '\\x01'
    > 'sink.properties.row_delimiter' = '\\x02'  
    > ```

   **处理同步数据**

   在同步过程中，如果您需要对数据进行一定的处理，例如 GROUP BY、JOIN 等，则可以修改 SQL 文件 **flink-create.all.sql**。本示例可以通过执行 count(*) 和 GROUP BY 计算出产品销量的实时排名。

   ```Bash
   $ ./bin/sql-client.sh -f flink-create.all.sql
   No default environment specified.
   Searching for '/home/disk1/flink-1.13.6/conf/sql-client-defaults.yaml'...not found.
   [INFO] Executing SQL from file.

   Flink SQL> CREATE DATABASE IF NOT EXISTS `default_catalog`.`demo`;
   [INFO] Execute statement succeed.

   -- 根据 MySQL 的订单表创建动态表 source table
   Flink SQL> 
   CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_src` (
     `order_id` BIGINT NOT NULL,
     `product_id` INT NULL,
     `order_date` TIMESTAMP NOT NULL,
     `customer_name` STRING NOT NULL,
     `product_name` STRING NOT NULL,
     `price` DECIMAL(10, 5) NULL,
     PRIMARY KEY(`order_id`)
    NOT ENFORCED
   ) with (
     'connector' = 'mysql-cdc',
     'hostname' = 'xxx.xx.xxx.xxx',
     'port' = '3306',
     'username' = 'root',
     'password' = '',
     'database-name' = 'demo',
     'table-name' = 'orders'
   );
   [INFO] Execute statement succeed.

   -- 创建动态表 sink table
   Flink SQL> 
   CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_sink` (
    `product_id` INT NOT NULL,
    `product_name` STRING NOT NULL,
    `sales_cnt` BIGINT NOT NULL,
    PRIMARY KEY(`product_id`)
   NOT ENFORCED
   ) with (
     'sink.max-retries' = '10',
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

   -- 执行查询，实现产品实时排行榜功能，查询不断更新 sink table，以反映 source table 上的更改
   Flink SQL> 
   INSERT INTO `default_catalog`.`demo`.`orders_sink` select product_id,product_name, count(*) as cnt from `default_catalog`.`demo`.`orders_src` group by product_id,product_name;
   [INFO] Submitting SQL update statement to the cluster...
   [INFO] SQL update statement has been successfully submitted to the cluster:
   Job ID: 5ae005c4b3425d8bb13fe660260a35da
   ```

   如果您只需要同步部分数据，例如支付时间在 2021 年 01 月 01 日之后的数据，则可以在 INSERT INTO SELECT 语句中使用 WHERE order_date >'2021-01-01' 设置过滤条件。不满足该条件的数据，即支付时间在 2021 年 01 月 01 日或者之前的数据不会同步至 StarRocks。

   ```sql
   INSERT INTO `default_catalog`.`demo`.`orders_sink` SELECT product_id,product_name, COUNT(*) AS cnt FROM `default_catalog`.`demo`.`orders_src` WHERE order_date >'2021-01-01 00:00:01' GROUP BY product_id,product_name;
   ```

   如果返回如下结果，则表示 Flink job 已经提交，开始同步全量和增量数据。

   ```SQL
   [INFO] Submitting SQL update statement to the cluster...
   [INFO] SQL update statement has been successfully submitted to the cluster:
   Job ID: 5ae005c4b3425d8bb13fe660260a35da
   ```

2. 可以通过 [Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui) 或者在 Flink 命令行执行命令`bin/flink list -running`，查看 Flink 集群中正在运行的 Flink job，以及 Flink job ID。
      1. Flink WebUI 界面
         ![task 拓扑](../assets/4.9.3.png)

      2. 在 Flink 命令行执行命令`bin/flink list -running`

         ```Bash
         $ bin/flink list -running
         Waiting for response...
         ------------------ Running/Restarting Jobs -------------------
         13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
         --------------------------------------------------------------
         ```

           > 说明
           >
           > 如果任务出现异常，可以通过 Flink WebUI 或者  **flink-1.14.5/log** 目录的日志文件进行排查。

## 常见问题

### **如何为不同的表设置不同的 flink-connector-starrocks 配置**

例如数据源某些表更新频繁，需要提高 flink connector sr 的导入速度等，则需要在 SMT 配置文件 **config_prod.conf** 中为这些表设置单独的 flink-connector-starrocks 配置。

```Bash
[table-rule.1]
# pattern to match databases for setting properties
database = ^order.*$
# pattern to match tables for setting properties
table = ^.*$

############################################
### flink sink configurations
### DO NOT set `connector`, `table-name`, `database-name`, they are auto-generated
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url= <fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=15000

[table-rule.2]
# pattern to match databases for setting properties
database = ^order2.*$
# pattern to match tables for setting properties
table = ^.*$

############################################
### flink sink configurations
### DO NOT set `connector`, `table-name`, `database-name`, they are auto-generated
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

### **同步 MySQL 分库分表后的多张表至 StarRocks 的一张表**

如果数据源 MySQL 进行分库分表，数据拆分成多张表甚至分布在多个库中，并且所有表的结构都是相同的，则您可以设置`[table-rule]`，将这些表同步至 StarRocks 的一张表中。比如 MySQL 有两个数据库 edu_db_1，edu_db_2，每个数据库下面分别有两张表 course_1，course_2，并且所有表的结构都是相同的，则通过设置如下`[table-rule]`可以将其同步至 StarRocks的一张表中。

> **说明**
>
> 数据源多张表同步至 StarRocks的一张表，表名默认为 course__auto_shard。如果需要修改，则可以在 **result** 目录的 SQL 文件  **starrocks-create.all.sql、 flink-create.all.sql** 中修改。

```Bash
[table-rule.1]
# pattern to match databases for setting properties
database = ^edu_db_[0-9]*$
# pattern to match tables for setting properties
table = ^course_[0-9]*$

############################################
### flink sink configurations
### DO NOT set `connector`, `table-name`, `database-name`, they are auto-generated
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

### **数据以 JSON 格式导入**

以上示例数据以 CSV 格式导入，如果数据无法选出合适的分隔符，则您需要替换 `[table-rule]` 中`flink.starrocks.*`的如下参数。

```Plain
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
```

传入如下参数，数据以 JSON 格式导入。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注意**
>
> 该方式会对导入速度有一定的影响。

### 多个的 INSERT INTO 语句合并为一个 Flink job

在 **flink-create.all.sql** 文件使用 [STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements) 语句，将多个的 INSERT INTO 语句合并为一个 Flink job，避免占用过多的 Flink job 资源。

   > 说明
   >
   > Flink 自 1.13 起 支持  [STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements) 语法。

1. 打开 **result/flink-create.all.sql** 文件。

2. 修改文件中的 SQL 语句，将所有的  INSERT INTO 语句调整位置到文件末尾。然后在第一条 INSERT语句的前面加上`EXECUTE STATEMENT SET BEGIN` 在最后一 INSERT 语句后面加上一行`END;`。

   > **注意**
   >
   > CREATE DATABASE、CREATE TABLE  的位置保持不变。

   ```SQL
   CREATE DATABASE IF NOT EXISTS db;
   CREATE TABLE IF NOT EXISTS db.a1;
   CREATE TABLE IF NOT EXISTS db.b1;
   CREATE TABLE IF NOT EXISTS db.a2;
   CREATE TABLE IF NOT EXISTS db.b2;
   EXECUTE STATEMENT SET 
   BEGIN
     -- 1个或者多个 INSERT INTO statements
   INSERT INTO db.a1 SELECT * FROM db.b1;
   INSERT INTO db.a2 SELECT * FROM db.b2;
   END;
   ```

更多常见问题，请参见 [MySQL 实时同步至 StarRocks 常见问题](../faq/loading/synchronize_mysql_into_sr.md)。