---
displayed_sidebar: English
---

# 从 MySQL 实现实时数据同步

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote

StarRocks 支持从 MySQL 实现秒级实时数据同步，能够以超低延迟实现大规模实时分析，支持用户实时查询数据。

本教程将帮助您了解如何为您的业务和用户提供实时分析。它演示了如何使用以下工具将 MySQL 中的数据实时同步到 StarRocks：StarRocks 迁移工具（SMT）、Flink、Flink CDC 连接器和 flink-starrocks-connector。

<InsertPrivNote />

## 工作原理

下图展示了整个同步过程。

![图片](../assets/4.9.2.png)

MySQL 的实时同步分为两个阶段：同步数据库和表结构，以及同步数据。首先，SMT 将 MySQL 数据库和表结构转换为 StarRocks 的建表语句。然后，Flink 集群运行 Flink 作业，将全量和增量的 MySQL 数据同步到 StarRocks。

> **注意**
>
> 同步过程保证了 exactly-once 语义。

**同步过程**：

1. 同步数据库和表结构。

   SMT 读取待同步的 MySQL 数据库和表的 schema，并生成 SQL 文件，用于在 StarRocks 中创建目标数据库和表。此操作基于 SMT 配置文件中的 MySQL 和 StarRocks 信息。

2. 同步数据。

   a. Flink SQL 客户端执行数据加载语句 `INSERT INTO SELECT`，以提交一个或多个 Flink 作业到 Flink 集群。

   b. Flink 集群通过 Flink 作业获取数据。[Flink CDC 连接器](https://github.com/ververica/flink-cdc-connectors/blob/master/docs/content/quickstart/build-real-time-data-lake-tutorial.md) 首先从源数据库读取完整的历史数据，然后无缝切换到增量读取，并将数据发送到 flink-starrocks-connector。

   c. flink-starrocks-connector 以小批量方式积累数据，并将每批数据同步到 StarRocks。

> **注意**
>
> 只有 MySQL 中的数据操作语言（DML）操作可以同步到 StarRocks。数据定义语言（DDL）操作无法同步。

## 应用场景

MySQL 的实时同步适用于数据不断变化的广泛用例。以真实世界的用例“商品销售实时排名”为例。

Flink 根据 MySQL 中原始订单表计算商品销售的实时排名，并将排名实时同步到 StarRocks 的主键表中。用户可以连接可视化工具到 StarRocks，实时查看排名，以获取即时的运营洞察。

## 准备工作

### 下载和安装同步工具

要从 MySQL 同步数据，您需要安装以下工具：SMT、Flink、Flink CDC 连接器和 flink-starrocks-connector。

1. 下载并安装 Flink，并启动 Flink 集群。您也可以按照[Flink 官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)中的说明执行此步骤。

   a. 在运行 Flink 之前，请在操作系统中安装 Java 8 或 Java 11。您可以运行以下命令来查看已安装的 Java 版本。

    ```Bash
        # 查看 Java 版本。
        java -version
        
        # 如果返回以下输出，则表示已安装 Java 8。
        java version "1.8.0_301"
        Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
        Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
    ```

   b. 下载[Flink 安装包](https://flink.apache.org/downloads.html)并解压。建议使用 Flink 1.14 或更高版本。允许的最低版本为 Flink 1.11。本主题使用的是 Flink 1.14.5。

   ```Bash
      # 下载 Flink。
      wget https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
      # 解压 Flink。  
      tar -xzf flink-1.14.5-bin-scala_2.11.tgz
      # 进入 Flink 目录。
      cd flink-1.14.5
    ```

   c. 启动 Flink 集群。

   ```Bash
      # 启动 Flink 集群。
      ./bin/start-cluster.sh
      
      # 如果返回以下输出，则表示 Flink 集群已启动。
      Starting cluster.
      Starting standalonesession daemon on host.
      Starting taskexecutor daemon on host.
    ```

2. 下载[Flink CDC 连接器](https://github.com/ververica/flink-cdc-connectors/releases)。本主题使用 MySQL 作为数据源，因此将下载 `flink-sql-connector-mysql-cdc-x.x.x.jar`。连接器版本必须与[Flink](https://github.com/ververica/flink-cdc-connectors/releases)版本匹配。有关详细的版本映射，请参见[支持的 Flink 版本](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions)。本主题使用的是 Flink 1.14.5，您可以下载 `flink-sql-connector-mysql-cdc-2.2.0.jar`。

    ```Bash
    wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.1.1/flink-sql-connector-mysql-cdc-2.2.0.jar
    ```

3. 下载[flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks)。版本必须与 Flink 版本匹配。

    > flink-connector-starrocks 软件包 `x.x.x_flink-y.yy _ z.zz.jar` 包含三个版本号：
    >
    > - `x.x.x` 是 flink-connector-starrocks 的版本号。
    > - `y.yy` 是支持的 Flink 版本。
    > - `z.zz` 是 Flink 支持的 Scala 版本。如果 Flink 版本为 1.14.x 或更早版本，则必须下载具有 Scala 版本的软件包。
    >
    > 本主题使用的是 Flink 1.14.5 和 Scala 2.11。因此，您可以下载以下软件包：`1.2.3_flink-14_2.11.jar`。

4. 将 Flink CDC 连接器（`flink-sql-connector-mysql-cdc-2.2.0.jar`）和 flink-connector-starrocks（`1.2.3_flink-1.14_2.11.jar`）的 JAR 包移动到 Flink 的 `lib` 目录下。

    > **注意**
    >
    > 如果您的系统中已经运行了 Flink 集群，则必须停止并重新启动 Flink 集群才能加载和验证 JAR 包。
    >
    > ```Bash
    > $ ./bin/stop-cluster.sh
    > $ ./bin/start-cluster.sh
    > ```

5. 下载并解压[SMT 包](https://www.starrocks.io/download/community)，并将其放入 `flink-1.14.5` 目录中。StarRocks 提供了适用于 Linux x86 和 macOS ARM64 的 SMT 包。您可以根据操作系统和 CPU 选择其中一个。

    ```Bash
    # 适用于 Linux x86
    wget https://releases.starrocks.io/resources/smt.tar.gz
    # 适用于 macOS ARM64
    wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
    ```

### 启用 MySQL 二进制日志

为了实现 MySQL 中的实时数据同步，系统需要读取 MySQL 二进制日志（binlog），解析数据，然后将数据同步到 StarRocks。请确保已启用 MySQL 二进制日志。

1. 编辑 MySQL 配置文件 `my.cnf`（默认路径：`/etc/my.cnf`），以启用 MySQL 二进制日志。

    ```Bash
    # 启用 MySQL Binlog。
    log_bin = ON
    # 配置 Binlog 的保存路径。
    log_bin = /var/lib/mysql/mysql-bin
    # 配置 server_id。
    # 如果未为 MySQL 5.7.3 或更高版本配置 server_id，则无法使用 MySQL 服务。
    server_id = 1
    # 将 Binlog 格式设置为 ROW。
    binlog_format = ROW
    # Binlog 文件的基本名称。每个 Binlog 文件都附加了标识符以标识每个 Binlog 文件。
    log_bin_basename = /var/lib/mysql/mysql-bin
    # Binlog 文件的索引文件，用于管理所有 Binlog 文件的目录。
    log_bin_index =/var/lib/mysql/mysql-bin.index
    ```

2. 运行以下命令之一以重新启动 MySQL，使修改后的配置文件生效。

    ```Bash
    # 使用 service 重新启动 MySQL。
    service mysqld restart
    # 使用 mysqld 脚本重新启动 MySQL。
    /etc/init.d/mysqld restart
    ```

3. 连接到 MySQL 并检查 MySQL 二进制日志是否已启用。

    ```Plain
    -- 连接到 MySQL。
    mysql -h xxx.xx.xxx.xx -P 3306 -u root -pxxxxxx

    -- 检查 MySQL 二进制日志是否已启用。
    mysql> SHOW VARIABLES LIKE 'log_bin'; 
    +---------------+-------+
    | Variable_name | Value |
    +---------------+-------+
    | log_bin       | ON    |
    +---------------+-------+
    1 row in set (0.00 sec)
    ```

## 同步数据库和表结构

1. 编辑 SMT 配置文件。
   进入 SMT `conf` 目录，编辑配置文件 `config_prod.conf`，包括 MySQL 连接信息、待同步库和表的匹配规则，以及 flink-starrocks-connector 的配置信息等。

    ```Bash
    [db]
    host = xxx.xx.xxx.xx
    port = 3306
    user = user1
    password = xxxxxx

    [other]
    # StarRocks 中的 BE 数量
    be_num = 3
    # StarRocks-1.18.1 版本开始支持 `decimal_v3`。
    use_decimal_v3 = true
    # 保存转换后的 DDL SQL 的文件
    output_dir = ./result

    [table-rule.1]
    # 用于设置属性的数据库匹配规则
    database = ^demo.*$
    # 用于设置属性的表匹配规则
    table = ^.*$

    ############################################
    ### Flink sink 配置
    ### 请勿设置 `connector`、`table-name`、`database-name`。它们是自动生成的。
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

    - `[db]`：MySQL 连接信息。
       - `host`：MySQL 服务器的 IP 地址。
       - `port`：MySQL 数据库的端口号，默认为 `3306`
       - `user`：用于访问 MySQL 数据库的用户名
       - `password`：用户名的密码

    - `[table-rule]`：数据库和表匹配规则以及相应的 flink-connector-starrocks 配置。

       - `Database`， `table`：MySQL 中数据库和表的名称。支持正则表达式。
       - `flink.starrocks.*`：flink-connector-starrocks 的配置信息。更多配置和信息，请参见 [flink-connector-starrocks](../loading/Flink-connector-starrocks.md)。

       > 如果需要对不同的表使用不同的 flink-connector-starrocks 配置。例如，如果某些表更新频繁，需要加速数据加载，请参见[不同表使用不同的 flink-connector-starrocks 配置](#use-different-flink-connector-starrocks-configurations-for-different-tables)。如果您需要将 MySQL 分片获取的多张表加载到同一个  StarRocks 表中，请参见 [MySQL 分片后将多张表同步到 StarRocks 中的一张表](#synchronize-multiple-tables-after-mysql-sharding-to-one-table-in-starrocks)中。

    - `[other]`： 其他信息
       - `be_num`：StarRocks 集群中的 BE 数量（该参数将用于在后续 StarRocks 表创建中设置合理的 tablet 数量）。
       - `use_decimal_v3`：是否启用 [Decimal V3](../sql-reference/sql-statements/data-types/DECIMAL.md)。开启 Decimal V3 后，MySQL 的 decimal 数据会在数据同步到 StarRocks 时转换为 Decimal V3 数据。
       - `output_dir`：保存要生成的 SQL 文件的路径。SQL 文件将用于在 StarRocks 中创建数据库和表，并将 Flink 作业提交到 Flink 集群。默认路径为 `./result` ，我们建议您保留默认设置。

2. 运行 SMT 读取 MySQL 中的数据库和表结构，并根据配置文件在 `./result` 目录中生成 SQL 文件。`starrocks-create.all.sql` 文件用于在 StarRocks 中创建数据库和表，`flink-create.all.sql` 文件用于向 Flink 集群提交 Flink 作业。

    ```Bash
    # 运行 SMT。
    ./starrocks-migrate-tool

    # 进入 result 目录并检查该目录中的文件。
    cd result
    ls result
    flink-create.1.sql    smt.tar.gz              starrocks-create.all.sql
    flink-create.all.sql  starrocks-create.1.sql
    ```

3. 运行以下命令连接到 StarRocks，并执行 `starrocks-create.all.sql` 文件，在 StarRocks 中创建数据库和表。建议您使用 SQL 文件中的默认表创建语句创建[主键表](../table_design/table_types/primary_key_table.md)。

    > **注意**
    >
    > 您也可以根据业务需要修改表创建语句，创建不使用主键表的表。但是，源 MySQL 数据库中的 DELETE 操作无法同步到非主键表。在创建此类表时要小心。

    ```Bash
    mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
    ```

    如果数据在写入目标 StarRocks 表之前需要经过 Flink 处理，则源表和目标表的表结构会有所不同。在这种情况下，您必须修改表创建语句。在此示例中，目标表仅需要 `product_id` `product_name` 商品销售的 and 列和实时排名。您可以使用以下表创建语句。

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
    > 从 v2.5.7 开始，StarRocks 可以在您创建表或添加分区时自动设置 BUCKET 数量。您不再需要手动设置存储桶数量。有关详细信息，请参阅 [确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

## 同步数据

运行 Flink 集群并提交 Flink 作业，将 MySQL 的全量数据和增量数据持续同步到 StarRocks。

1. 进入 Flink 目录，并运行以下命令在 Flink SQL 客户端上运行 `flink-create.all.sql` 文件。

    ```Bash
    ./bin/sql-client.sh -f flink-create.all.sql
    ```

    此 SQL 文件定义动态表 `source table` 和 `sink table` 查询语句 `INSERT INTO SELECT`，并指定连接器、源数据库和目标数据库。执行该文件后，将 Flink 作业提交到 Flink 集群，开始数据同步。

    > **注意**
    >
    > - 确保 Flink 集群已启动。您可以通过执行以下命令来启动 Flink 集群 `flink/bin/start-cluster.sh`。
    > - 如果您的 Flink 版本低于 1.13，您可能无法直接运行 SQL 文件 `flink-create.all.sql`。您需要在 SQL 客户端的命令行界面 （CLI） 中逐个执行此文件中的 SQL 语句。你还需要转义 `\` 这个角色。
    >
    > ```Bash
    > 'sink.properties.column_separator' = '\\x01'
    > 'sink.properties.row_delimiter' = '\\x02'  
    > ```

    **同步期间的处理数据**：

    如果需要在同步过程中处理数据，例如对数据执行 GROUP BY 或 JOIN，则可以修改 `flink-create.all.sql` 文件。以下示例通过执行 COUNT （*） 和 GROUP BY 来计算商品销售的实时排名。

    ```Bash
        $ ./bin/sql-client.sh -f flink-create.all.sql
        No default environment is specified.
        Searching for '/home/disk1/flink-1.13.6/conf/sql-client-defaults.yaml'...not found.
        [INFO] Executing SQL from file.

        Flink SQL> CREATE DATABASE IF NOT EXISTS `default_catalog`.`demo`;
        [INFO] Execute statement succeed.


        -- 基于 MySQL 中的订单表创建动态表 `source table`。
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
        [INFO] 执行语句成功。

        -- 创建动态表 `sink table`。
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
        [INFO] 执行语句成功。

        -- 实现商品销售的实时排名，其中 `sink table` 动态更新以反映 `source table` 中的数据更改。
        Flink SQL> 
        INSERT INTO `default_catalog`.`demo`.`orders_sink` select product_id,product_name, count(*) as cnt from `default_catalog`.`demo`.`orders_src` group by product_id,product_name;
        [INFO] 提交 SQL 更新语句到集群...
        [INFO] SQL 更新语句已成功提交到集群：
        作业 ID: 5ae005c4b3425d8bb13fe660260a35da
    ```

    如果只需要同步部分数据，例如支付时间晚于 2021 年 12 月 21 日的数据，则可以使用 `INSERT INTO SELECT` 中的 `WHERE` 子句设置筛选条件，例如 `WHERE pay_dt > '2021-12-21'`。不满足此条件的数据将不会同步到 StarRocks。

    如果返回结果如下，则表示 Flink 作业已提交，进行全量同步和增量同步。

    ```SQL
    [INFO] 提交 SQL 更新语句到集群...
    [INFO] SQL 更新语句已成功提交到集群：
    作业 ID: 5ae005c4b3425d8bb13fe660260a35da
    ```

2. 您可以使用 [Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui) 或在 Flink SQL 客户端上运行 `bin/flink list -running` 命令来查看在 Flink 集群中正在运行的 Flink 作业和作业 ID。

    - Flink WebUI
      ![img](../assets/4.9.3.png)

    - `bin/flink list -running`

    ```Bash
        $ bin/flink list -running
        等待响应...
        ------------------ 正在运行/重新启动的作业 -------------------
        13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
        --------------------------------------------------------------
    ```

    > **注意**
    >
    > 如果作业异常，您可以通过 Flink WebUI 或查看 Flink 1.14.5 目录下的 `/log` 目录中的日志文件进行故障排除。

## 常见问题

### 对不同的表使用不同的 flink-connector-starrocks 配置

如果数据源中的一些表经常更新，并且您希望加快 flink-connector-starrocks 的加载速度，则必须在 SMT 配置文件 `config_prod.conf` 中为每个表设置单独的 flink-connector-starrocks 配置。

```Bash
[table-rule.1]
# 用于设置属性的数据库匹配模式
database = ^order.*$
# 用于设置属性的表匹配模式
table = ^.*$

############################################
### Flink sink 配置
### 请勿设置 `connector`、`table-name`、`database-name`，它们是自动生成的
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url= <fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=15000[table-rule.2]
# 用于设置属性的数据库匹配模式
database = ^order2.*$
# 用于设置属性的表匹配模式
table = ^.*$

############################################
### Flink sink 配置
### 请勿设置 `connector`、`table-name`、`database-name`，它们是自动生成的
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

### 在 StarRocks 中将 MySQL 分片后的多张表同步为一张表

执行分片后，一个 MySQL 表中的数据可能会被拆分成多个表，甚至分发到多个数据库。所有表具有相同的架构。在这种情况下，您可以设置 `[table-rule]` 将所有这些表同步到一个 StarRocks 表中。例如，MySQL 有两个数据库 `edu_db_1` 和 `edu_db_2`，每个数据库有两个表 `course_1` 和 `course_2`，并且所有表的架构都相同。您可以使用以下 `[table-rule]` 配置将所有表同步到一个 StarRocks 表中。

> **注意**
>
> StarRocks 表的名称默认为 `course__auto_shard`。如果需要使用其他名称，可以在 SQL 文件 `starrocks-create.all.sql` 和 `flink-create.all.sql` 中进行修改。

```Bash
[table-rule.1]
# 用于设置属性的数据库匹配模式
database = ^edu_db_[0-9]*$
# 用于设置属性的表匹配模式
table = ^course_[0-9]*$

############################################
### Flink sink 配置
### 请勿设置 `connector`、`table-name`、`database-name`，它们是自动生成的
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

上述示例中的数据以 CSV 格式导入。如果无法选择合适的分隔符，则需要替换 `[table-rule]` 中 `flink.starrocks.*` 的以下参数。

```Plain
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
```

传入以下参数后，以 JSON 格式导入数据。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注意**
>
> 这种方法会略微降低加载速度。

### 将多个 INSERT INTO 语句作为一个 Flink 作业执行

您可以使用 [STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements) 语法在 `flink-create.all.sql` 文件中执行多个 INSERT INTO 语句作为一个 Flink 作业，这样可以防止多个语句占用过多的 Flink 作业资源，并提高执行多个查询的效率。

> **注意**
>
> 从 1.13 版本开始，Flink 支持 STATEMENT SET 语法。

1. 打开 `result/flink-create.all.sql` 文件。

2. 修改文件中的 SQL 语句。将所有 INSERT INTO 语句移到文件末尾。在第一个 INSERT INTO 语句之前放置 `EXECUTE STATEMENT SET BEGIN`，在最后一个 INSERT INTO 语句之后放置 `END;`。

> **注意**
> CREATE DATABASE 和 CREATE TABLE 的位置保持不变。

```SQL
IF NOT EXISTS db 创建数据库;
IF NOT EXISTS db.a1 创建表;
IF NOT EXISTS db.b1 创建表;
IF NOT EXISTS db.a2 创建表;
IF NOT EXISTS db.b2 创建表;
执行语句设置
开始-- 一个或多个插入语句
INSERT INTO db.a1 SELECT * FROM db.b1;
INSERT INTO db.a2 SELECT * FROM db.b2;
END;