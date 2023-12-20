---
displayed_sidebar: English
---

# 通过 Audit Loader 管理 StarRocks 中的审计日志

本主题介绍如何通过插件 - Audit Loader 在表中管理 StarRocks 的审计日志。

StarRocks 将其审计日志存储在本地文件 **fe/log/fe.audit.log** 中，而不是在内部数据库。插件 Audit Loader 允许您直接在集群内管理审计日志。Audit Loader 从文件中读取日志，并通过 HTTP PUT 请求将其加载到 StarRocks 中。

## 创建一个表来存储审计日志

在 StarRocks 集群中创建一个数据库和一张表来存储其审计日志。详细说明请参见 [CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) 和 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

由于不同版本的 StarRocks 审计日志字段各不相同，您必须根据以下示例来创建一个与您的 StarRocks 版本兼容的表。

> **注意**
> 请勿更改示例中的表结构，或日志加载将失败。

- 适用于 StarRocks v2.4、v2.5、v3.0、v3.1 及后续小版本的示例：

```SQL
CREATE DATABASE starrocks_audit_db__;

CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "Unique query ID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "Query start time",
  `queryType`      VARCHAR(12)            COMMENT "Query type (query, slow_query)",
  `clientIp`       VARCHAR(32)            COMMENT "Client IP address",
  `user`           VARCHAR(64)            COMMENT "User who initiates the query",
  `authorizedUser` VARCHAR(64)            COMMENT "user_identity",
  `resourceGroup`  VARCHAR(64)            COMMENT "Resource group name",
  `catalog`        VARCHAR(32)            COMMENT "Catalog name",
  `db`             VARCHAR(96)            COMMENT "Database that the query scans",
  `state`          VARCHAR(8)             COMMENT "Query state (EOF, ERR, OK)",
  `errorCode`      VARCHAR(96)            COMMENT "Error code",
  `queryTime`      BIGINT                 COMMENT "Query latency in milliseconds",
  `scanBytes`      BIGINT                 COMMENT "Size of the scanned data in bytes",
  `scanRows`       BIGINT                 COMMENT "Row count of the scanned data",
  `returnRows`     BIGINT                 COMMENT "Row count of the result",
  `cpuCostNs`      BIGINT                 COMMENT "CPU resources consumption time for query in nanoseconds",
  `memCostBytes`   BIGINT                 COMMENT "Memory cost for query in bytes",
  `stmtId`         INT                    COMMENT "Incremental SQL statement ID",
  `isQuery`        TINYINT                COMMENT "If the SQL is a query (0 and 1)",
  `feIp`           VARCHAR(32)            COMMENT "IP address of FE that executes the SQL",
  `stmt`           STRING                 COMMENT "SQL statement",
  `digest`         VARCHAR(32)            COMMENT "SQL fingerprint",
  `planCpuCosts`   DOUBLE                 COMMENT "CPU resources consumption time for planning in nanoseconds",
  `planMemCosts`   DOUBLE                 COMMENT "Memory cost for planning in bytes"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `queryType`)
COMMENT "Audit log table"
PARTITION BY RANGE (`timestamp`) ()
DISTRIBUTED BY HASH (`queryId`) BUCKETS 3 
PROPERTIES (
  "dynamic_partition.time_unit" = "DAY",
  "dynamic_partition.start" = "-30",
  "dynamic_partition.end" = "3",
  "dynamic_partition.prefix" = "p",
  "dynamic_partition.buckets" = "3",
  "dynamic_partition.enable" = "true",
  "replication_num" = "3"
);
```

- 适用于 StarRocks v2.3.0 及后续小版本的示例：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "Unique query ID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "Query start time",
  `clientIp`       VARCHAR(32)            COMMENT "Client IP address",
  `user`           VARCHAR(64)            COMMENT "User who initiates the query",
  `resourceGroup`  VARCHAR(64)            COMMENT "Resource group name",
  `db`             VARCHAR(96)            COMMENT "Database that the query scans",
  `state`          VARCHAR(8)             COMMENT "Query state (EOF, ERR, OK)",
  `errorCode`      VARCHAR(96)            COMMENT "Error code",
  `queryTime`      BIGINT                 COMMENT "Query latency in milliseconds",
  `scanBytes`      BIGINT                 COMMENT "Size of the scanned data in bytes",
  `scanRows`       BIGINT                 COMMENT "Row count of the scanned data",
  `returnRows`     BIGINT                 COMMENT "Row count of the result",
  `cpuCostNs`      BIGINT                 COMMENT "CPU resources consumption time for query in nanoseconds",
  `memCostBytes`   BIGINT                 COMMENT "Memory cost for query in bytes",
  `stmtId`         INT                    COMMENT "Incremental SQL statement ID",
  `isQuery`        TINYINT                COMMENT "If the SQL is a query (0 and 1)",
  `feIp`           VARCHAR(32)            COMMENT "IP address of FE that executes the SQL",
  `stmt`           STRING                 COMMENT "SQL statement",
  `digest`         VARCHAR(32)            COMMENT "SQL fingerprint",
  `planCpuCosts`   DOUBLE                 COMMENT "CPU resources consumption time for planning in nanoseconds",
  `planMemCosts`   DOUBLE                 COMMENT "Memory cost for planning in bytes"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `clientIp`)
COMMENT "Audit log table"
PARTITION BY RANGE (`timestamp`) ()
DISTRIBUTED BY HASH (`queryId`) BUCKETS 3 
PROPERTIES (
  "dynamic_partition.time_unit" = "DAY",
  "dynamic_partition.start" = "-30",
  "dynamic_partition.end" = "3",
  "dynamic_partition.prefix" = "p",
  "dynamic_partition.enable" = "true",
  "replication_num" = "3"
);
```

- 适用于 StarRocks v2.2.1 及后续小版本的示例：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id         VARCHAR(48)            COMMENT "Unique query ID",
    time             DATETIME     NOT NULL  COMMENT "Query start time",
    client_ip        VARCHAR(32)            COMMENT "Client IP address",
    user             VARCHAR(64)            COMMENT "User who initiates the query",
    db               VARCHAR(96)            COMMENT "Database that the query scans",
    state            VARCHAR(8)             COMMENT "Query state (EOF, ERR, OK)",
    query_time       BIGINT                 COMMENT "Query latency in milliseconds",
    scan_bytes       BIGINT                 COMMENT "Size of the scanned data in bytes",
    scan_rows        BIGINT                 COMMENT "Row count of the scanned data",
    return_rows      BIGINT                 COMMENT "Row count of the result",
    cpu_cost_ns      BIGINT                 COMMENT "CPU resources consumption time for query in nanoseconds",
    mem_cost_bytes   BIGINT                 COMMENT "Memory cost for query in bytes",
    stmt_id          INT                    COMMENT "Incremental SQL statement ID",
    is_query         TINYINT                COMMENT "If the SQL is a query (0 and 1)",
    frontend_ip      VARCHAR(32)            COMMENT "IP address of FE that executes the SQL",
    stmt             STRING                 COMMENT "SQL statement",
    digest           VARCHAR(32)            COMMENT "SQL fingerprint"
) engine=OLAP
DUPLICATE KEY(query_id, time, client_ip)
PARTITION BY RANGE(time) ()
DISTRIBUTED BY HASH(query_id) BUCKETS 3 
PROPERTIES(
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-30",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.enable" = "true",
    "replication_num" = "3"
);
```

- 适用于 StarRocks v2.2.0、v2.1.0 及后续小版本的示例：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "Unique query ID",
    time            DATETIME    NOT NULL  COMMENT "Query start time",
    client_ip       VARCHAR(32)           COMMENT "Client IP address",
    user            VARCHAR(64)           COMMENT "User who initiates the query",
    db              VARCHAR(96)           COMMENT "Database that the query scans",
    state           VARCHAR(8)            COMMENT "Query state (EOFE, RR, OK)",
    query_time      BIGINT                COMMENT "Query latency in milliseconds",
    scan_bytes      BIGINT                COMMENT "Size of the scanned data in bytes",
    scan_rows       BIGINT                COMMENT "Row count of the scanned data",
    return_rows     BIGINT                COMMENT "Row count of the result",
    stmt_id         INT                   COMMENT "Incremental SQL statement ID",
    is_query        TINYINT               COMMENT "If the SQL is a query (0 and 1)",
    frontend_ip     VARCHAR(32)           COMMENT "IP address of FE that executes the SQL",
    stmt            STRING                COMMENT "SQL statement",
    digest          VARCHAR(32)           COMMENT "SQL fingerprint"
) engine=OLAP
DUPLICATE KEY(query_id, time, client_ip)
PARTITION BY RANGE(time) ()
DISTRIBUTED BY HASH(query_id) BUCKETS 3 
PROPERTIES(
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-30",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.enable" = "true",
    "replication_num" = "3"
);
```

- 适用于 StarRocks v2.0.0 及后续小版本、StarRocks v1.19.0 及后续小版本的示例：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "Unique query ID",
    time            DATETIME    NOT NULL  COMMENT "Query start time",
    client_ip       VARCHAR(32)           COMMENT "Client IP address",
    user            VARCHAR(64)           COMMENT "User who initiates the query",
    db              VARCHAR(96)           COMMENT "Database that the query scans",
    state           VARCHAR(8)            COMMENT "Query state (EOF, ERR, OK)",
    query_time      BIGINT                COMMENT "Query latency in milliseconds",
    scan_bytes      BIGINT                COMMENT "Size of the scanned data in bytes",
    scan_rows       BIGINT                COMMENT "Row count of the scanned data",
    return_rows     BIGINT                COMMENT "Row count of the result",
    stmt_id         INT                   COMMENT "Incremental SQL statement ID",
    is_query        TINYINT               COMMENT "If the SQL is a query (0 and 1)",
    frontend_ip     VARCHAR(32)           COMMENT "IP address of FE that executes the SQL",
    stmt            STRING                COMMENT "SQL statement"
) engine=OLAP
DUPLICATE KEY(query_id, time, client_ip)
PARTITION BY RANGE(time) ()
DISTRIBUTED BY HASH(query_id) BUCKETS 3 
PROPERTIES(
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-30",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.enable" = "true",
    "replication_num" = "3"
);
```

starrocks_audit_tbl__ 表是使用动态分区创建的。默认情况下，第一个动态分区会在表创建后 10 分钟内生成。之后，审计日志便可以加载到该表中。您可以使用以下语句来检查表中的分区情况：

```SQL
SHOW PARTITIONS FROM starrocks_audit_db__.starrocks_audit_tbl__;
```

分区创建后，您可以进行下一步。

## 下载并配置 Audit Loader

1. 下载[安装包](https://releases.starrocks.io/resources/AuditLoader.zip)。该包含了适用于不同 StarRocks 版本的多个目录。您需要导航至相应的目录并安装与您的 StarRocks 版本兼容的包。

   -  **2.4**：适用于 StarRocks v2.4.0 及后续小版本
   -  **2.3**：适用于 StarRocks v2.3.0 及后续小版本
   -  **2.2.1+**：适用于 StarRocks v2.2.1及后续小版本
   -  **2.1-2.2.0**：适用于 StarRocks v2.2.0、StarRocks v2.1.0 及后续小版本
   -  **1.18.2-2.0**：适用于 StarRocks v2.0.0 及后续小版本、StarRocks v1.19.0 及后续小版本

2. 解压安装包。

   ```shell
   unzip auditloader.zip
   ```

   解压后的文件包括：

   -  **auditloader.jar**：Audit Loader 的 JAR 文件。
   -  **plugin.properties**：Audit Loader 的属性文件。
   -  **plugin.conf**：Audit Loader 的配置文件。

3. 修改 **plugin.conf** 以配置 Audit Loader。您必须配置以下项目以确保 Audit Loader 正常工作：

   -  frontend_host_port：FE IP 地址和 HTTP 端口，格式为 <fe_ip>:<fe_http_port>。默认值是 127.0.0.1:8030。
   -  database：您创建用来存储审计日志的数据库的名称。
   -  table：您创建用来存储审计日志的表的名称。
   -  user：您的集群用户名。您必须拥有向表中加载数据的权限（LOAD_PRIV）。
   -  password：您的用户密码。

4. 将文件重新打包。

   ```shell
   zip -q -m -r auditloader.zip auditloader.jar plugin.conf plugin.properties
   ```

5. 将打包好的文件分发到托管 FE 节点的所有机器上。确保所有包存放在相同的路径下，否则安装将失败。分发包后，请记下包的绝对路径。

## 安装 Audit Loader

执行以下命令，并附上您记下的路径，以在 StarRocks 中安装 Audit Loader 插件：

```SQL
INSTALL PLUGIN FROM "<absolute_path_to_package>";
```

查看[INSTALL PLUGIN](../sql-reference/sql-statements/Administration/INSTALL_PLUGIN.md)获取详细指南。

## 验证安装并查询审计日志

1. 您可以通过[SHOW PLUGINS](../sql-reference/sql-statements/Administration/SHOW_PLUGINS.md)命令来检查安装是否成功。

   在以下示例中，AuditLoader 插件的状态为 INSTALLED，表明安装成功。

   ```Plain
   mysql> SHOW PLUGINS\G
   *************************** 1. row ***************************
       Name: __builtin_AuditLogBuilder
       Type: AUDIT
   Description: builtin audit logger
       Version: 0.12.0
   JavaVersion: 1.8.31
   ClassName: com.starrocks.qe.AuditLogBuilder
       SoName: NULL
       Sources: Builtin
       Status: INSTALLED
   Properties: {}
   *************************** 2. row ***************************
       Name: AuditLoader
       Type: AUDIT
   Description: load audit log to olap load, and user can view the statistic of queries
       Version: 1.0.1
   JavaVersion: 1.8.0
   ClassName: com.starrocks.plugin.audit.AuditLoaderPlugin
       SoName: NULL
       Sources: /x/xx/xxx/xxxxx/auditloader.zip
       Status: INSTALLED
   Properties: {}
   2 rows in set (0.01 sec)
   ```

2. 执行一些随机的 SQL 语句以生成审计日志，并等待 60 秒（或您在配置 Audit Loader 时指定的 max_batch_interval_sec 参数所设置的时间），以便 Audit Loader 将审计日志加载到 StarRocks 中。

3. 通过查询表来检查审计日志。

   ```SQL
   SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__;
   ```

   以下示例展示了审计日志成功加载到表中的情况：

   ```Plain
   mysql> SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__\G
   *************************** 1. row ***************************
       queryId: 082ddf02-6492-11ed-a071-6ae6b1db20eb
       timestamp: 2022-11-15 11:03:08
       clientIp: xxx.xx.xxx.xx:33544
           user: root
   resourceGroup: default_wg
               db: 
           state: EOF
       errorCode: 
       queryTime: 8
       scanBytes: 0
       scanRows: 0
       returnRows: 0
       cpuCostNs: 62380
   memCostBytes: 14504
           stmtId: 33
       isQuery: 1
           feIp: xxx.xx.xxx.xx
           stmt: SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__
           digest: 
   planCpuCosts: 21
   planMemCosts: 0
   1 row in set (0.01 sec)
   ```

## 故障排除

如果在创建动态分区和安装插件后，表中没有加载审计日志，您可以检查是否正确配置**plugin.conf**。要修改配置，您需要先卸载插件：

```SQL
UNINSTALL PLUGIN AuditLoader;
```

配置正确后，您可以按照上述步骤重新安装 Audit Loader。
