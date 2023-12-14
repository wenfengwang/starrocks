# 实时从MySQL进行数据同步

导入InsertPrivNote自'../assets/commonMarkdown/insertPrivNote.md'

StarRocks支持从MySQL实时同步数据，可以在几秒内进行数据分析，并且可以让用户在数据发生时进行查询。

本教程将帮助您了解如何为您的业务和用户引入实时分析。它演示了如何使用以下工具实现实时将数据从MySQL同步到StarRocks：StarRocks迁移工具（SMT）、Flink、Flink CDC连接器和flink-starrocks连接器。

<InsertPrivNote />

## 工作原理

以下图示了整个同步过程。

![img](../assets/4.9.2.png)

从MySQL实时同步数据分为两个阶段：同步数据库和表结构，以及同步数据。首先， SMT将MySQL数据库和表结构转换为StarRocks的表创建语句。然后，Flink集群运行Flink作业将完整和增量的MySQL数据同步到StarRocks。

> **注意**

>

> 同步过程保证精确一次语义。

**同步过程**：

1. 同步数据库和表结构。

   SMT读取要同步的MySQL数据库和表的结构，并生成用于在StarRocks中创建目标数据库和表的SQL文件。此操作基于SMT配置文件中的MySQL和StarRocks信息。

2. 同步数据。

   a. Flink SQL客户端执行数据加载语句 `INSERT INTO SELECT`，提交一个或多个Flink作业到Flink集群。

   b. Flink集群运行Flink作业以获取数据。[Flink CDC连接器](https://github.com/ververica/flink-cdc-connectors/blob/master/docs/content/quickstart/build-real-time-data-lake-tutorial.md)首先从源数据库中读取完整的历史数据，然后无缝切换到增量读取，并将数据发送到flink-starrocks连接器。

   c. flink-starrocks连接器积累数据到迷你批次，并将每个数据批次同步到StarRocks。

> **注意**

>
> 仅能够同步MySQL中的数据操纵语言（DML）操作到StarRocks。 数据定义语言（DDL）操作不能被同步。


## 场景

从MySQL进行实时数据同步具有广泛的使用案例，其中数据不断发生变化。以“商品销售的实时排名”为例。

Flink根据MySQL中的原始订单表计算商品销售的实时排名，并实时同步排名到StarRocks的主键表中。用户可以连接可视化工具到StarRocks以实时查看排名，获取按需的运营洞察。

## 准备工作

### 下载和安装同步工具

要从MySQL同步数据，您需要安装以下工具：SMT、Flink、Flink CDC连接器和flink-starrocks连接器。

1. 下载并安装Flink，并启动Flink集群。您也可以按照[Flink官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/)中的说明进行操作。

   a. 在运行Flink之前，请在您的操作系统中安装Java 8或Java 11。您可以运行以下命令检查已安装的Java版本。

    ```Bash
        # 查看Java版本。
        java -version
        
        # 如果返回以下输出，则表示已安装Java 8。
        java version "1.8.0_301"
        Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
        Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
    ```

   b. 下载[Flink安装包](https://flink.apache.org/downloads.html)并解压缩。我们建议您使用Flink 1.14或更新版本。最低允许版本为Flink 1.11。本主题使用Flink 1.14.5。

   ```Bash
      # 下载Flink。
      wget https://archive.apache.org/dist/flink/flink-1.14.5/flink-1.14.5-bin-scala_2.11.tgz
      # 解压缩Flink。
      tar -xzf flink-1.14.5-bin-scala_2.11.tgz
      # 进入Flink目录。
      cd flink-1.14.5
    ```

   c. 启动Flink集群。

   ```Bash
      # 启动Flink集群。
      ./bin/start-cluster.sh
      
      # 如果返回以下输出，则表示已启动Flink集群。
      Starting cluster.
      Starting standalonesession daemon on host.
      Starting taskexecutor daemon on host.
    ```

2. 下载[Flink CDC连接器](https://github.com/ververica/flink-cdc-connectors/releases)。本主题使用MySQL作为数据源，因此下载`flink-sql-connector-mysql-cdc-x.x.x.jar`。连接器版本必须与[Flink](https://github.com/ververica/flink-cdc-connectors/releases)版本匹配。有关详细的版本映射，请参见[支持的Flink版本](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions)。本主题使用Flink 1.14.5，您可以下载`flink-sql-connector-mysql-cdc-2.2.0.jar`。

    ```Bash
    wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.1.1/flink-sql-connector-mysql-cdc-2.2.0.jar
    ```

3. 下载[flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks)。版本必须与Flink版本匹配。

    > flink-connector-starrocks包`x.x.x_flink-y.yy _ z.zz.jar`包含三个版本号：
    >
    > - `x.x.x` 是flink-connector-starrocks的版本号。
    > - `y.yy` 是Flink的支持版本。
    > - `z.zz` 是Flink支持的Scala版本。如果Flink版本是1.14.x或更早，您必须下载具有Scala版本的包。
    >
    > 本主题使用Flink 1.14.5和Scala 2.11。因此，您可以下载以下软件包：`1.2.3_flink-14_2.11.jar`。

4. 将Flink CDC连接器的JAR包（`flink-sql-connector-mysql-cdc-2.2.0.jar`）和flink-connector-starrocks的JAR包（`1.2.3_flink-1.14_2.11.jar`）移动到Flink的`lib`目录中。

    > **注意**
    >
    > 如果系统中已经运行了Flink集群，则必须停止Flink集群并重新启动以加载和验证JAR包。
    >
    > ```Bash
    > $ ./bin/stop-cluster.sh
    > $ ./bin/start-cluster.sh
    > ```

5. 下载并解压[SMT包](https://www.starrocks.io/download/community)，并将其放置在`flink-1.14.5`目录中。StarRocks提供Linux x86和macos ARM64的SMT包。您可以根据您的操作系统和CPU选择一个包。

    ```Bash
    # 对于Linux x86
    wget https://releases.starrocks.io/resources/smt.tar.gz
    # 对于macOS ARM64
    wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
    ```

### 启用MySQL二进制日志

为了实时从MySQL同步数据，系统需要读取MySQL二进制日志（binlog）的数据，并将数据解析然后同步到StarRocks。确保MySQL二进制日志已启用。

1. 编辑MySQL配置文件`my.cnf`（默认路径：`/etc/my.cnf`）以启用MySQL二进制日志。

    ```Bash
    # 启用MySQL Binlog。
    log_bin = ON
    # 配置Binlog的保存路径。
    log_bin =/var/lib/mysql/mysql-bin
    # 配置server_id。
    # 如果未为MySQL 5.7.3或更高版本配置server_id，则无法使用MySQL服务。
    server_id = 1
    # 将Binlog格式设置为ROW。
    binlog_format = ROW
    # Binlog文件的基本名称。每个Binlog文件都会附加一个标识符以标识该文件。
    log_bin_basename =/var/lib/mysql/mysql-bin
    # Binlog文件的索引文件，用于管理所有Binlog文件的目录。
    log_bin_index =/var/lib/mysql/mysql-bin.index
    ```

2. 运行以下命令之一，以重新启动MySQL使修改后的配置文件生效。

    ```Bash
# 使用服务来重启 MySQL。
    service mysqld restart
# 使用 mysqld 脚本来重启 MySQL。
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
   进入 SMT `conf` 目录并编辑配置文件 `config_prod.conf`，例如 MySQL 连接信息，要同步的数据库和表的匹配规则，以及 flink-starrocks-connector 的配置信息。

    ```Bash
    [db]
    host = xxx.xx.xxx.xx
    port = 3306
    user = user1
    password = xxxxxx

    [other]
    # StarRocks 中的 BE 数量
    be_num = 3
    # `decimal_v3` 在 StarRocks-1.18.1 版本开始支持。
    use_decimal_v3 = true
    # 保存转换后的 DDL SQL 的文件路径
    output_dir = ./result

    [table-rule.1]
    # 匹配数据库以设置属性的模式
    database = ^demo.*$
    # 匹配表以设置属性的模式
    table = ^.*$

    ############################################
    ### Flink sink 配置
    ### 不要设置 `connector`、`table-name`、`database-name`。它们是自动生成的。
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
   - `port`：MySQL 数据库的端口号，默认为 `3306`。
   - `user`：访问 MySQL 数据库的用户名。
   - `password`：用户名的密码。

- `[table-rule]`：数据库和表匹配规则以及对应的 flink-connector-starrocks 配置信息。

   - `Database`、`table`：MySQL 中数据库和表的名称。支持正则表达式。
   - `flink.starrocks.*`：flink-connector-starrocks 的配置信息。更多配置和信息，请参阅 [flink-connector-starrocks](../loading/Flink-connector-starrocks.md)。

   > 如果需要为不同的表使用不同的 flink-connector-starrocks 配置。例如，如果某些表经常更新，需要加速数据加载，请参阅[为不同的表使用不同的 flink-connector-starrocks 配置](#use-different-flink-connector-starrocks-configurations-for-different-tables)。如果需要将从 MySQL 切分中获取的多个表加载到同一张 StarRocks 表中，请参阅[将 MySQL 切分后的多个表同步到 StarRocks 中的一张表](#synchronize-multiple-tables-after-MySQL-sharding-to-one-table-in-StarRocks)。

- `[other]`：其他信息
   - `be_num`：StarRocks 集群中 BE 的数量（此参数将用于设置后续在 StarRocks 表创建中合理的分片数量）。
   - `use_decimal_v3`：是否启用 [Decimal V3](../sql-reference/sql-statements/data-types/DECIMAL.md)。启用 Decimal V3 后，同步到 StarRocks 时，MySQL 的 decimal 数据将被转换为 Decimal V3 数据。
   - `output_dir`：要生成的 SQL 文件的保存路径。SQL 文件将用于在 StarRocks 中创建数据库和表，并向 Flink 集群提交 Flink 作业。默认路径为 `./result`，建议保留默认设置。

2. 运行 SMT 从 MySQL 中读取数据库和表结构，并基于配置文件在 `./result` 目录下生成 SQL 文件。`starrocks-create.all.sql` 文件用于在 StarRocks 中创建数据库和表，`flink-create.all.sql` 文件用于向 Flink 集群提交 Flink 作业。

    ```Bash
    # 运行 SMT。
    ./starrocks-migrate-tool

    # 进入 result 目录并检查此目录中的文件。
    cd result
    ls result
    flink-create.1.sql    smt.tar.gz              starrocks-create.all.sql
    flink-create.all.sql  starrocks-create.1.sql
    ```

3. 运行以下命令连接到 StarRocks 并执行 `starrocks-create.all.sql` 文件，以在 StarRocks 中创建数据库和表。我们建议您使用 SQL 文件中的默认表创建语句创建[主键表](../table_design/table_types/primary_key_table.md)。

    > **注意**
    >
    > 您也可以根据业务需求修改表创建语句，并创建不使用主键表的表。然而，源 MySQL 数据库中的删除操作无法同步到非主键表。在创建这样的表时，请谨慎操作。

    ```Bash
    mysql -h <fe_host> -P <fe_query_port> -u user2 -pxxxxxx < starrocks-create.all.sql
    ```

    如果在将数据写入目标 StarRocks 表之前需要由 Flink 进行处理，则源表和目标表之间的表结构将不同。在这种情况下，您必须修改表创建语句。在此示例中，目标表仅需要 `product_id` 和 `product_name` 列以及商品销售的实时排行。您可以使用以下表创建语句。

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
    > 自 v2.5.7 版本以来，StarRocks 在创建表或添加分区时可以自动设置桶数（BUCKETS），您无需再手动设置桶数。有关详细信息，请参阅[确定桶数](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

## 同步数据

运行 Flink 集群并提交一个 Flink 作业，以持续地将 MySQL 中的全量和增量数据同步到 StarRocks 中。

1. 进入 Flink 目录并运行以下命令，在 Flink SQL 客户端上运行 `flink-create.all.sql` 文件。

    ```Bash
    ./bin/sql-client.sh -f flink-create.all.sql
    ```

    此 SQL 文件定义了动态表 `source table` 和 `sink table`、查询语句 `INSERT INTO SELECT`，并指定连接器、源数据库和目标数据库。执行此文件后，将向 Flink 集群提交一个 Flink 作业，开始数据同步。

    > **注意**
    >
    > - 请确保 Flink 集群已经启动。您可以通过运行 `flink/bin/start-cluster.sh` 来启动 Flink 集群。
    > - 如果您的 Flink 版本早于 1.13，可能无法直接运行 SQL 文件 `flink-create.all.sql`。您需要在 SQL 客户端的命令行界面（CLI）中逐个执行此文件中的 SQL 语句，并且需要转义 `\` 字符。
    >
    > ```Bash
    > 'sink.properties.column_separator' = '\\x01'
    > 'sink.properties.row_delimiter' = '\\x02'  
    > ```

    **同步过程中的数据处理**：

    如果您需要在同步过程中处理数据，例如对数据进行 GROUP BY 或 JOIN，您可以修改 `flink-create.all.sql` 文件。以下示例根据需要执行 COUNT(*) 和 GROUP BY 来实现商品销售的实时排行。

    ```Bash
    $ ./bin/sql-client.sh -f flink-create.all.sql
    No default environment is specified.
    Searching for '/home/disk1/flink-1.13.6/conf/sql-client-defaults.yaml'...not found.
    [INFO] Executing SQL from file.

    Flink SQL> CREATE DATABASE IF NOT EXISTS `default_catalog`.`demo`;
    [INFO] Execute statement succeed.

-- 基于MySQL中的订单表创建`source table`动态表。
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

-- 创建`sink table`动态表。
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

-- 实现商品销售的实时排名，`sink table`动态更新以反映`source table`中数据更改。
Flink SQL> 
INSERT INTO `default_catalog`.`demo`.`orders_sink` select product_id,product_name, count(*) as cnt from `default_catalog`.`demo`.`orders_src` group by product_id,product_name;
[INFO] 向集群提交SQL更新语句...
[INFO] SQL更新语句已成功提交到集群：
Job ID: 5ae005c4b3425d8bb13fe660260a35da

如果只需同步部分数据，例如付款时间晚于2021年12月21日的数据，您可以在 `INSERT INTO SELECT` 中使用 `WHERE` 子句设置过滤条件，例如 `WHERE pay_dt > '2021-12-21'`。 不满足此条件的数据将不会同步到 StarRocks。

如果返回以下结果，则已向 Flink 作业提交了完整和增量同步。

```SQL
[INFO] 向集群提交SQL更新语句...
[INFO] SQL更新语句已成功提交到集群：
Job ID: 5ae005c4b3425d8bb13fe660260a35da
```

2. 您可以使用[Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui) 或在 Flink SQL 客户端上运行 `bin/flink list -running` 命令来查看 Flink 集群中正在运行的 Flink 作业和作业ID。

    - Flink WebUI
      ![img](../assets/4.9.3.png)

    - `bin/flink list -running`

    ```Bash
        $ bin/flink list -running
        等待响应...
        ------------------ 正在运行/重启的作业 -------------------
        13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
        --------------------------------------------------------------
    ```

    > **注意**
    >
    > 如果作业异常，您可以使用 Flink WebUI 进行故障排除，或者查看 Flink 1.14.5 的 `/log` 目录中的日志文件。

## 常见问题

### 为不同的表使用不同的 flink-connector-starrocks 配置

如果数据源中的某些表经常更新，并且要加速 flink-connector-starrocks 的加载速度，必须为 SMT 配置文件 `config_prod.conf` 中的每个表设置一个单独的 flink-connector-starrocks 配置。

```Bash
[table-rule.1]
# 匹配数据库以设置属性
database = ^order.*$
# 匹配表以设置属性
table = ^.*$

############################################
### Flink sink 配置
### 不要设置 `connector`、`table-name`、`database-name`。它们是自动生成的
############################################
flink.starrocks.jdbc-url=jdbc:mysql://<fe_host>:<fe_query_port>
flink.starrocks.load-url= <fe_host>:<fe_http_port>
flink.starrocks.username=user2
flink.starrocks.password=xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator=\x01
flink.starrocks.sink.properties.row_delimiter=\x02
flink.starrocks.sink.buffer-flush.interval-ms=15000[table-rule.2]
# 匹配数据库以设置属性
database = ^order2.*$
# 匹配表以设置属性
table = ^.*$

############################################
### Flink sink 配置
### 不要设置 `connector`、`table-name`、`database-name`。它们是自动生成的
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

### 将MySQL分片后的多表同步至 StarRocks 的一个表

执行分片后，一个MySQL表中的数据可能被分割为多个表，甚至分布到多个数据库中。所有的表具有相同的模式。在这种情况下，您可以设置 `[table-rule]` 将这些表同步到一个 StarRocks 表中。例如，MySQL 有两个数据库 `edu_db_1` 和 `edu_db_2`，每个数据库都有两个表 `course_1 和 course_2`，并且所有表的模式都相同。您可以使用以下 `[table-rule]` 配置将所有表同步到一个 StarRocks 表中。

> **注意**
>
> StarRocks 表的名称默认为 `course__auto_shard`。如果您需要使用不同的名称，可以在 SQL 文件 `starrocks-create.all.sql` 和 `flink-create.all.sql` 中进行修改。

```Bash
[table-rule.1]
# 匹配数据库以设置属性
database = ^edu_db_[0-9]*$
# 匹配表以设置属性
table = ^course_[0-9]*$

############################################
### Flink sink 配置
### 不要设置 `connector`、`table-name`、`database-name`。它们是自动生成的
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

### 以JSON格式导入数据

上面示例中的数据是以CSV格式导入的。如果您无法选择适当的分隔符，您需要在`[table-rule]`中替换 `flink.starrocks.*` 的以下参数。

```Plain
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
```

在传递以下参数后，数据是以JSON格式导入的。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注意**
>
> 此方法稍微减慢了加载速度。

### 将多个 INSERT INTO 语句作为一个 Flink 作业执行

您可以在`flink-create.all.sql`文件中使用[STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements)语法，将多个 INSERT INTO 语句作为一个 Flink 作业执行，这样可以防止多个语句占用过多的 Flink 作业资源，并提高执行多个查询的效率。

> **注意**
```SQL
从1.13版开始，Flink支持STATEMENT SET语法。

1. 打开`result/flink-create.all.sql`文件。

2. 修改文件中的SQL语句。将所有的INSERT INTO语句移动到文件末尾。在第一个INSERT INTO语句之前放置`EXECUTE STATEMENT SET BEGIN`，在最后一个INSERT INTO语句之后放置`END;`。

> **注意**
>
> CREATE DATABASE和CREATE TABLE的位置保持不变。

```SQL
CREATE DATABASE IF NOT EXISTS db;
CREATE TABLE IF NOT EXISTS db.a1;
CREATE TABLE IF NOT EXISTS db.b1;
CREATE TABLE IF NOT EXISTS db.a2;
CREATE TABLE IF NOT EXISTS db.b2;
EXECUTE STATEMENT SET 
BEGIN-- 一个或多个INSERT INTO语句
INSERT INTO db.a1 SELECT * FROM db.b1;
INSERT INTO db.a2 SELECT * FROM db.b2;
END;
```