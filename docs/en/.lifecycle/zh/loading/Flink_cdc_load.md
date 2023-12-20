---
displayed_sidebar: English
---

# 从 MySQL 实时同步

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks 支持从 MySQL 在数秒内进行实时数据同步，提供超低延迟的大规模实时分析，并使用户能够查询实时发生的数据。

本教程将帮助您了解如何为您的企业和用户带来实时分析。它演示了如何使用以下工具将数据从 MySQL 实时同步到 StarRocks：StarRocks 迁移工具（SMT）、Flink、Flink CDC 连接器和 flink-starrocks-connector。

<InsertPrivNote />


## 工作原理

下图展示了整个同步过程。

![img](../assets/4.9.2.png)

从 MySQL 的实时同步分为两个阶段实现：同步数据库和表架构以及同步数据。首先，SMT 将 MySQL 数据库和表架构转换为 StarRocks 的表创建语句。然后，Flink 集群运行 Flink 作业以实时同步 MySQL 的全量和增量数据到 StarRocks。

> **注意**
> 同步过程保证精确一次语义。

**同步过程**：

1. 同步数据库和表架构。

   SMT 读取要同步的 MySQL 数据库和表的架构，并生成 SQL 文件，用于在 StarRocks 中创建目标数据库和表。此操作基于 SMT 配置文件中的 MySQL 和 StarRocks 信息。

2. 同步数据。

   a. Flink SQL 客户端执行数据加载语句 `INSERT INTO SELECT`，向 Flink 集群提交一个或多个 Flink 作业。

   b. Flink 集群运行 Flink 作业来获取数据。[Flink CDC 连接器](https://github.com/ververica/flink-cdc-connectors/blob/master/docs/content/quickstart/build-real-time-data-lake-tutorial.md) 首先从源数据库读取完整的历史数据，然后无缝切换到增量读取，并将数据发送到 flink-starrocks-connector。

   c. flink-starrocks-connector 累积小批量数据，并将每批数据同步到 StarRocks。

> **注意**
> 只有 MySQL 中的数据操作语言（DML）操作可以同步到 StarRocks。数据定义语言（DDL）操作无法同步。

## 应用场景

MySQL 的实时同步适用于数据不断变化的广泛用例。以现实世界的用例“商品销售实时排名”为例。

Flink 根据 MySQL 中的原始订单表计算商品销售的实时排名，并实时将排名同步到 StarRocks 的主键表中。用户可以将可视化工具连接到 StarRocks，实时查看排名，以获得按需运营洞察。

## 准备工作

### 下载并安装同步工具

要从 MySQL 同步数据，您需要安装以下工具：SMT、Flink、Flink CDC 连接器和 flink-starrocks-connector。

1. 下载并安装 Flink，并启动 Flink 集群。您也可以按照 [Flink 官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.13/docs/try-flink/local_installation/) 的说明执行此步骤。

   a. 在运行 Flink 之前，请在操作系统中安装 Java 8 或 Java 11。您可以运行以下命令来检查已安装的 Java 版本。

   ```Bash
       # 查看 Java 版本。
       java -version
   
       # 如果返回以下输出，则表示已安装 Java 8。
       java version "1.8.0_301"
       Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
       Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
   ```

   b. 下载 [Flink 安装包](https://flink.apache.org/downloads.html) 并解压。我们建议您使用 Flink 1.14 或更高版本。允许的最低版本是 Flink 1.11。本主题使用 Flink 1.14.5。

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

2. 下载 [Flink CDC 连接器](https://github.com/ververica/flink-cdc-connectors/releases)。本主题使用 MySQL 作为数据源，因此下载 `flink-sql-connector-mysql-cdc-x.x.x.jar`。连接器版本必须与 [Flink](https://github.com/ververica/flink-cdc-connectors/releases) 版本匹配。有关详细的版本映射，请参见 [支持的 Flink 版本](https://ververica.github.io/flink-cdc-connectors/release-2.2/content/about.html#supported-flink-versions)。本主题使用 Flink 1.14.5，您可以下载 `flink-sql-connector-mysql-cdc-2.2.0.jar`。

   ```Bash
   wget https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.1.1/flink-sql-connector-mysql-cdc-2.2.0.jar
   ```

3. 下载 [flink-connector-starrocks](https://search.maven.org/artifact/com.starrocks/flink-connector-starrocks)。版本必须与 Flink 版本匹配。

      > flink-connector-starrocks 包 `x.x.x_flink-y.yy_z.zz.jar` 包含三个版本号：
   - `x.x.x` 是 flink-connector-starrocks 的版本号。
   - `y.yy` 是支持的 Flink 版本。
   - `z.zz` 是 Flink 支持的 Scala 版本。如果 Flink 版本是 1.14.x 或更早版本，您必须下载包含 Scala 版本的包。
      > 本主题使用 Flink 1.14.5 和 Scala 2.11。因此，您可以下载以下包：`1.2.3_flink-1.14_2.11.jar`。

4. 将 Flink CDC 连接器（`flink-sql-connector-mysql-cdc-2.2.0.jar`）和 flink-connector-starrocks（`1.2.3_flink-1.14_2.11.jar`）的 JAR 包移动到 Flink 的 `lib` 目录下。

      > **注意**
      > 如果您的系统中已经运行了 Flink 集群，您必须停止 Flink 集群并重新启动它，以加载和验证 JAR 包。
   ```Bash
   $ ./bin/stop-cluster.sh
   $ ./bin/start-cluster.sh
   ```

5. 下载并解压 [SMT 包](https://www.starrocks.io/download/community)，并将其放在 `flink-1.14.5` 目录下。StarRocks 为 Linux x86 和 macOS ARM64 提供了 SMT 包。您可以根据您的操作系统和 CPU 选择一个。

   ```Bash
   # for Linux x86
   wget https://releases.starrocks.io/resources/smt.tar.gz
   # for macOS ARM64
   wget https://releases.starrocks.io/resources/smt_darwin_arm64.tar.gz
   ```

### 启用 MySQL 二进制日志

为了实时同步 MySQL 数据，系统需要从 MySQL 二进制日志（binlog）读取数据，解析数据，然后将数据同步到 StarRocks。确保已启用 MySQL 二进制日志。

1. 编辑 MySQL 配置文件 `my.cnf`（默认路径：`/etc/my.cnf`）以启用 MySQL 二进制日志。

   ```Bash
   # 启用 MySQL Binlog。
   log_bin = ON
   # 配置 Binlog 的保存路径。
   log_bin =/var/lib/mysql/mysql-bin
   # 配置 server_id。
   # 如果 MySQL 5.7.3 或更高版本未配置 server_id，MySQL 服务将无法使用。
   server_id = 1
   # 将 Binlog 格式设置为 ROW。
   binlog_format = ROW
   # Binlog 文件的基本名称。一个标识符被追加以识别每个 Binlog 文件。
   log_bin_basename =/var/lib/mysql/mysql-bin
```
-- 创建一个名为 `source table` 的动态表，基于MySQL中的订单表。
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
   
       -- 创建一个名为 `sink table` 的动态表。
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
   
       -- 实现商品销量的实时排名，`sink table` 动态更新以反映 `source table` 中的数据变化。
       Flink SQL> 
       INSERT INTO `default_catalog`.`demo`.`orders_sink` select product_id,product_name, count(*) as cnt from `default_catalog`.`demo`.`orders_src` group by product_id,product_name;
       [INFO] Submitting SQL update statement to the cluster...
       [INFO] SQL update statement has been successfully submitted to the cluster:
       Job ID: 5ae005c4b3425d8bb13fe660260a35da
   ```

   如果只需要同步部分数据，例如支付时间晚于2021年12月21日的数据，可以在`INSERT INTO SELECT`中使用`WHERE`子句设置过滤条件，例如`WHERE pay_dt > '2021-12-21'`。不满足此条件的数据将不会同步到StarRocks。

   如果返回以下结果，则表示已提交Flink作业进行全量和增量同步。

   ```SQL
   [INFO] Submitting SQL update statement to the cluster...
   [INFO] SQL update statement has been successfully submitted to the cluster:
   Job ID: 5ae005c4b3425d8bb13fe660260a35da
   ```

2. 您可以使用[Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui)或在Flink SQL客户端上运行`bin/flink list -running`命令来查看在Flink集群中正在运行的Flink作业和作业ID。

-     Flink WebUI
![img](../assets/4.9.3.png)

-     `bin/flink list -running`

   ```Bash
       $ bin/flink list -running
       Waiting for response...
       ------------------ Running/Restarting Jobs -------------------
       13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
       --------------------------------------------------------------
   ```

      > **注意**
      > 如果作业异常，您可以使用Flink WebUI进行故障排除，或查看Flink 1.14.5的`/log`目录中的日志文件。

## 常见问题

### 对不同表使用不同的 flink-connector-starrocks 配置

如果数据源中的某些表经常更新，并且您希望加速 flink-connector-starrocks 的加载速度，您必须为SMT配置文件`config_prod.conf`中的每个表设置单独的 flink-connector-starrocks 配置。

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

### 将MySQL分片后的多张表同步到StarRocks中的一张表

在分片后，一个MySQL表中的数据可能被拆分为多个表，甚至分布在多个数据库中。所有表的架构相同。在这种情况下，您可以设置`[table-rule]`，将这些表同步到一个StarRocks表中。例如，MySQL有两个数据库`edu_db_1`和`edu_db_2`，每个数据库中有两个表`course_1`和`course_2`，所有表的架构相同。您可以使用以下`[table-rule]`配置，将所有表同步到一个StarRocks表中。

> **注意**
> StarRocks表的名称默认为`course__auto_shard`。如果需要使用不同的名称，可以在SQL文件`starrocks-create.all.sql`和`flink-create.all.sql`中进行修改。

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

### 以JSON格式导入数据

前面的示例中以CSV格式导入数据。如果无法选择合适的分隔符，您需要替换`[table-rule]`中的以下`flink.starrocks.*`参数。

```Plain
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
```

将以下参数传入，以JSON格式导入数据。

```Plain
flink.starrocks.sink.properties.format=json
flink.starrocks.sink.properties.strip_outer_array=true
```

> **注意**
> 这种方法会稍微降低加载速度。

### 将多个INSERT INTO语句作为一个Flink作业执行

您可以在`flink-create.all.sql`文件中使用[STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements)语法将多个INSERT INTO语句作为一个Flink作业执行，防止多个语句占用过多的Flink作业资源，提高执行多个查询的效率。

> **注意**
> Flink 从 1.13 版本开始支持 STATEMENT SET 语法。

1. 打开`result/flink-create.all.sql`文件。

2. 修改文件中的SQL语句，将所有的INSERT INTO语句移到文件末尾。在第一个INSERT INTO语句之前放置`EXECUTE STATEMENT SET BEGIN`，在最后一个INSERT INTO语句之后放置`END;`。

> **注意**
> CREATE DATABASE 和 CREATE TABLE 的位置保持不变。

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
```
```SQL
-- 基于 MySQL 中的 order 表创建动态表 `source table`。
Flink SQL>
CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_src` (
  `order_id` BIGINT NOT NULL,
  `product_id` INT NULL,
  `order_date` TIMESTAMP NOT NULL,
  `customer_name` STRING NOT NULL,
  `product_name` STRING NOT NULL,
  `price` DECIMAL(10, 5) NULL,
  PRIMARY KEY(`order_id`) NOT ENFORCED
) WITH (
  'connector' = 'mysql-cdc',
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
CREATE TABLE IF NOT EXISTS `default_catalog`.`demo`.`orders_sink` (
  `product_id` INT NOT NULL,
  `product_name` STRING NOT NULL,
  `sales_cnt` BIGINT NOT NULL,
  PRIMARY KEY(`product_id`) NOT ENFORCED
) WITH (
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
[INFO] 执行语句成功。

-- 实现商品销售实时排名，`sink table` 动态反映 `source table` 中的数据变化。
Flink SQL>
INSERT INTO `default_catalog`.`demo`.`orders_sink` SELECT product_id, product_name, COUNT(*) AS cnt FROM `default_catalog`.`demo`.`orders_src` GROUP BY product_id, product_name;
[INFO] 正在向集群提交 SQL 更新语句...
[INFO] SQL 更新语句已成功提交到集群：
Job ID: 5ae005c4b3425d8bb13fe660260a35da
```

如果您只需要同步部分数据，例如支付时间晚于2021年12月21日的数据，可以在 `INSERT INTO SELECT` 中使用 `WHERE` 子句设置过滤条件，例如 `WHERE pay_dt > '2021-12-21'`。不满足此条件的数据将不会同步到 StarRocks。

如果返回如下结果，则说明 Flink 作业已提交进行全量和增量同步。

```SQL
[INFO] 正在向集群提交 SQL 更新语句...
[INFO] SQL 更新语句已成功提交到集群：
Job ID: 5ae005c4b3425d8bb13fe660260a35da
```

2. 您可以使用 [Flink WebUI](https://nightlies.apache.org/flink/flink-docs-master/docs/try-flink/flink-operations-playground/#flink-webui) 或在您的 Flink SQL 客户端上运行 `bin/flink list -running` 命令来查看 Flink 集群中正在运行的 Flink 作业和作业 ID。

- Flink WebUI
![img](../assets/4.9.3.png)

- `bin/flink list -running`

```Bash
$ bin/flink list -running
Waiting for response...
------------------ Running/Restarting Jobs -------------------
13.10.2022 15:03:54 : 040a846f8b58e82eb99c8663424294d5 : insert-into_default_catalog.lily.example_tbl1_sink (RUNNING)
--------------------------------------------------------------
```

> **注意**
> 如果作业异常，您可以通过 Flink WebUI 或查看 Flink 1.14.5 的 `/log` 目录下的日志文件进行排查。

## 常见问题解答

### 对不同的表使用不同的 flink-connector-starrocks 配置

如果数据源中的某些表更新频繁，并且您希望加快 flink-connector-starrocks 的加载速度，则必须在 SMT 配置文件 `config_prod.conf` 中为每个表设置单独的 flink-connector-starrocks 配置。

```Bash
[table-rule.1]
# 用于设置属性的数据库匹配模式
database = ^order.*$
# 用于设置属性的表匹配模式
table = ^.*$

############################################
### Flink sink 配置
### 不要设置 `connector`, `table-name`, `database-name`。它们会自动生成
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
# 用于设置属性的数据库匹配模式
database = ^order2.*$
# 用于设置属性的表匹配模式
table = ^.*$

############################################
### Flink sink 配置
### 不要设置 `connector`, `table-name`, `database-name`。它们会自动生成
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

### 将 MySQL 分片后的多张表同步到 StarRocks 中的一张表

分片后，一张 MySQL 表中的数据可能会被拆分为多张表，甚至分布到多个数据库中。所有表都具有相同的架构。此时，您可以设置 `[table-rule]` 将这些表同步到一张 StarRocks 表上。例如，MySQL 有两个数据库 `edu_db_1` 和 `edu_db_2`，每个数据库都有两个表 `course_1` 和 `course_2`，并且所有表的架构相同。您可以使用以下 `[table-rule]` 配置将所有表同步到一张 StarRocks 表。

> **注意**
> StarRocks 表的名称默认为 `course__auto_shard`。如果您需要使用不同的名称，可以在 SQL 文件 `starrocks-create.all.sql` 和 `flink-create.all.sql` 中进行修改。

```Bash
[table-rule.1]
# 用于设置属性的数据库匹配模式
database = ^edu_db_[0-9]*$
# 用于设置属性的表匹配模式
table = ^course_[0-9]*$

############################################
### Flink sink 配置
### 不要设置 `connector`, `table-name`, `database-name`。它们会自动生成
############################################
flink.starrocks.jdbc-url = jdbc:mysql://xxx.xxx.x.x:xxxx
flink.starrocks.load-url = xxx.xxx.x.x:xxxx
flink.starrocks.username = user2
flink.starrocks.password = xxxxxx
flink.starrocks.sink.properties.format=csv
flink.starrocks.sink.properties.column_separator =\x01
flink.starrocks.sink.properties.row_delimiter =\x02
flink.starrocks.sink.buffer-flush.interval-ms = 5000
```

### 以 JSON 格式导入数据

上例中的数据以 CSV 格式导入。如果您无法选择合适的分隔符，则需要替换 `[table-rule]` 中 `flink.starrocks.*` 的以下参数。

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
> 此方法会稍微减慢加载速度。

### 将多个 INSERT INTO 语句作为一个 Flink 作业执行

您可以在 `flink-create.all.sql` 文件中使用 [STATEMENT SET](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sqlclient/#execute-a-set-of-sql-statements) 语法执行多个 INSERT INTO 语句作为一个 Flink 作业，这样可以避免多条语句占用过多的 Flink 作业资源，提高执行多个查询的效率。

> **注意**
> Flink 从 1.13 版本开始支持 STATEMENT SET 语法。

1. 打开 `result/flink-create.all.sql` 文件。
```
```SQL
CREATE DATABASE IF NOT EXISTS db;
CREATE TABLE IF NOT EXISTS db.a1;
CREATE TABLE IF NOT EXISTS db.b1;
CREATE TABLE IF NOT EXISTS db.a2;
CREATE TABLE IF NOT EXISTS db.b2;
EXECUTE STATEMENT SET 
BEGIN -- 一个或多个 INSERT INTO 语句
INSERT INTO db.a1 SELECT * FROM db.b1;
INSERT INTO db.a2 SELECT * FROM db.b2;
END;
```