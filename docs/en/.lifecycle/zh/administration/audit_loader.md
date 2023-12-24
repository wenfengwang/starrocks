---
displayed_sidebar: English
---

# 通过 Audit Loader 管理 StarRocks 中的审计日志

本主题描述如何通过插件 - Audit Loader 在表内管理 StarRocks 审计日志。

StarRocks 将其审计日志存储在本地文件 **fe/log/fe.audit.log** 中，而不是内部数据库中。插件 Audit Loader 允许您直接在集群中管理审计日志。Audit Loader 从文件中读取日志，并通过 HTTP PUT 将其加载到 StarRocks 中。

## 创建表以存储审计日志

在 StarRocks 集群中创建数据库和表，以存储其审计日志。有关详细说明，请参阅 [CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) 和 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

由于不同 StarRocks 版本的审计日志字段存在差异，因此您必须从以下示例中进行选择，以创建与您的 StarRocks 兼容的表。

> **注意**
>
> 请勿更改示例中的表模式，否则日志加载将失败。

- StarRocks v2.4、v2.5、v3.0、v3.1 及以后的小版本：

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

- StarRocks v2.3.0 及以后的小版本：

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

- StarRocks v2.2.1 及以后的小版本：

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

- StarRocks v2.2.0、v2.1.0 及以后的小版本：

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
    return_rows     BIGINT                COMMENT "结果行数",
    stmt_id         INT                   COMMENT "增量 SQL 语句 ID",
    is_query        TINYINT               COMMENT "SQL 是否为查询（0 和 1）",
    frontend_ip     VARCHAR(32)           COMMENT "执行 SQL 的 FE 的 IP 地址",
    stmt            STRING                COMMENT "SQL 语句",
    digest          VARCHAR(32)           COMMENT "SQL 指纹"
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

- StarRocks v2.0.0 及更高版本，StarRocks v1.19.0 及更高版本：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "唯一查询 ID",
    time            DATETIME    NOT NULL  COMMENT "查询开始时间",
    client_ip       VARCHAR(32)           COMMENT "客户端 IP 地址",
    user            VARCHAR(64)           COMMENT "发起查询的用户",
    db              VARCHAR(96)           COMMENT "查询扫描的数据库",
    state           VARCHAR(8)            COMMENT "查询状态（EOF、ERR、OK）",
    query_time      BIGINT                COMMENT "查询延迟时间（毫秒）",
    scan_bytes      BIGINT                COMMENT "扫描数据的大小（字节）",
    scan_rows       BIGINT                COMMENT "扫描数据的行数",
    return_rows     BIGINT                COMMENT "结果行数",
    stmt_id         INT                   COMMENT "增量 SQL 语句 ID",
    is_query        TINYINT               COMMENT "SQL 是否为查询（0 和 1）",
    frontend_ip     VARCHAR(32)           COMMENT "执行 SQL 的 FE 的 IP 地址",
    stmt            STRING                COMMENT "SQL 语句"
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

`starrocks_audit_tbl__` 使用动态分区创建。默认情况下，表创建后 10 分钟后会创建第一个动态分区。然后，可以将审核日志加载到表中。您可以使用以下语句检查表中的分区：

```SQL
SHOW PARTITIONS FROM starrocks_audit_db__.starrocks_audit_tbl__;
```

创建分区后，可以继续执行下一步。

## 下载并配置审核加载程序

1. [下载](https://releases.starrocks.io/resources/AuditLoader.zip) 审核加载程序安装包。该软件包包含多个不同 StarRocks 版本的目录。您必须导航到相应的目录，并安装与您的 StarRocks 兼容的软件包。

    - **2.4**： StarRocks v2.4.0 及更高版本
    - **2.3**： StarRocks v2.3.0 及更高版本
    - **2.2.1+**：StarRocks v2.2.1及更高版本
    - **2.1-2.2.0**：StarRocks v2.2.0、StarRocks v2.1.0及更高版本
    - **1.18.2-2.0**：StarRocks v2.0.0及更高版本，StarRocks v1.19.0及更高版本

2. 解压安装包。

    ```shell
    unzip auditloader.zip
    ```

    解压后的文件如下：

    - **auditloader.jar**：Audit Loader 的 JAR 文件。
    - **plugin.properties**：Audit Loader 的属性文件。
    - **plugin.conf**：Audit Loader 的配置文件。

3. 修改 **plugin.conf** 以配置 Audit Loader。您必须配置以下项目，以确保 Audit Loader 可以正常工作：

    - `frontend_host_port`：FE IP 地址和 HTTP 端口，格式为 `<fe_ip>:<fe_http_port>`。默认值为 `127.0.0.1:8030`。
    - `database`：您创建的用于托管审核日志的数据库的名称。
    - `table`：您创建的用于托管审核日志的表的名称。
    - `user`：您的集群用户名。您必须具有将数据 （LOAD_PRIV） 加载到表中的权限。
    - `password`：您的用户密码。

4. 将文件重新打包成一个软件包。

    ```shell
    zip -q -m -r auditloader.zip auditloader.jar plugin.conf plugin.properties
    ```

5. 将软件包分发到托管 FE 节点的所有机器。确保所有软件包都存储在相同的路径中。否则，安装将失败。分发软件包后，请记住复制软件包的绝对路径。

## 安装审核加载程序

执行以下语句，并附上您复制的路径，将 Audit Loader 作为 StarRocks 中的插件安装：

```SQL
INSTALL PLUGIN FROM "<absolute_path_to_package>";
```

有关详细说明，请参阅 [INSTALL PLUGIN](../sql-reference/sql-statements/Administration/INSTALL_PLUGIN.md)。

## 验证安装并查询审核日志

1. 您可以通过 [SHOW PLUGINS](../sql-reference/sql-statements/Administration/SHOW_PLUGINS.md) 检查安装是否成功。

    在以下示例中，插件 `AuditLoader` 的 `Status` 是 `INSTALLED`，表示安装成功。

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

2. 执行一些随机的 SQL 来生成审核日志，并等待 60 秒（或您在配置 Audit Loader 时在项目中指定的时间 `max_batch_interval_sec` ），让 Audit Loader 将审核日志加载到 StarRocks 中。

3. 通过查询表来检查审核日志。

    ```SQL
    SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__;
    ```

    以下示例显示了审核日志何时成功加载到表中：

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

如果在创建动态分区并安装插件后，表中没有加载任何审计日志，您可以检查 **plugin.conf** 是否已正确配置。若需修改配置，您必须首先卸载插件：

```SQL
UNINSTALL PLUGIN AuditLoader;
```

在确认所有配置均已正确设置后，您可以按照上述步骤重新安装审计加载程序。