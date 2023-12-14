---
displayed_sidebar: "Chinese"
---

# Manage audit logs in StarRocks through Audit Loader

This document describes how to manage audit logs internally in StarRocks through the Audit Loader plugin.

StarRocks stores all audit logs in the local file **fe/log/fe.audit.log**, which cannot be accessed through the internal database system. The Audit Loader plugin allows you to manage audit logs within the cluster. Audit Loader can read logs from local files and import them into StarRocks via HTTP PUT.

## Create the audit log library table

Create a database and table for audit logs in the StarRocks cluster. For detailed operation instructions, see [CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) and [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md).

Because the audit log fields in different versions of StarRocks are different, the corresponding table creation statements are also different. You need to select the table creation statement corresponding to your version of the StarRocks cluster from the following examples.

> **Note**
>
> Do not modify the table properties in the example, otherwise it will cause the log import to fail.

- StarRocks v2.4, v2.5, v3.0, v3.1, and subsequent minor versions:

```SQL
CREATE DATABASE starrocks_audit_db__;

CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "Unique ID of the query",
  `timestamp`      DATETIME     NOT NULL  COMMENT "Query start time",
  `queryType`      VARCHAR(12)            COMMENT "Query type (query, slow_query)",
  `clientIp`       VARCHAR(32)            COMMENT "Client IP",
  `user`           VARCHAR(64)            COMMENT "Query username",
  `authorizedUser` VARCHAR(64)            COMMENT "User unique identifier, i.e. user_identity",
  `resourceGroup`  VARCHAR(64)            COMMENT "Resource group name",
  `catalog`        VARCHAR(32)            COMMENT "Catalog name",
  `db`             VARCHAR(96)            COMMENT "Database where the query is located",
  `state`          VARCHAR(8)             COMMENT "Query status (EOF, ERR, OK)",
  `errorCode`      VARCHAR(96)            COMMENT "Error code",
  `queryTime`      BIGINT                 COMMENT "Query execution time (milliseconds)",
  `scanBytes`      BIGINT                 COMMENT "Bytes scanned in the query",
  `scanRows`       BIGINT                 COMMENT "Record rows scanned in the query",
  `returnRows`     BIGINT                 COMMENT "Result rows returned in the query",
  `cpuCostNs`      BIGINT                 COMMENT "Query CPU time (nanoseconds)",
  `memCostBytes`   BIGINT                 COMMENT "Memory consumption for the query (bytes)",
  `stmtId`         INT                    COMMENT "SQL statement incremental ID",
  `isQuery`        TINYINT                COMMENT "Whether the SQL is a query (1 or 0)",
  `feIp`           VARCHAR(32)            COMMENT "FE IP that executed the statement",
  `stmt`           STRING                 COMMENT "Original SQL statement",
  `digest`         VARCHAR(32)            COMMENT "SQL fingerprint",
  `planCpuCosts`   DOUBLE                 COMMENT "CPU costs during query planning phase (nanoseconds)",
  `planMemCosts`   DOUBLE                 COMMENT "Memory costs during query planning phase (bytes)"
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
  "dynamic_partition.enable" = "true",
  "replication_num" = "3"
);
```

- StarRocks v2.3.0 and subsequent minor versions:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "Unique ID of the query",
  `timestamp`      DATETIME     NOT NULL  COMMENT "Query start time",
  `clientIp`       VARCHAR(32)            COMMENT "Client IP",
  `user`           VARCHAR(64)            COMMENT "Query username",
  `resourceGroup`  VARCHAR(64)            COMMENT "Resource group name",
  `db`             VARCHAR(96)            COMMENT "Database where the query is located",
  `state`          VARCHAR(8)             COMMENT "Query status (EOF, ERR, OK)",
  `errorCode`      VARCHAR(96)            COMMENT "Error code",
  `queryTime`      BIGINT                 COMMENT "Query execution time (milliseconds)",
  `scanBytes`      BIGINT                 COMMENT "Bytes scanned in the query",
  `scanRows`       BIGINT                 COMMENT "Record rows scanned in the query",
  `returnRows`     BIGINT                 COMMENT "Result rows returned in the query",
  `cpuCostNs`      BIGINT                 COMMENT "Query CPU time (nanoseconds)",
  `memCostBytes`   BIGINT                 COMMENT "Memory consumption for the query (bytes)",
  `stmtId`         INT                    COMMENT "SQL statement incremental ID",
  `isQuery`        TINYINT                COMMENT "Whether the SQL is a query (1 or 0)",
  `feIp`           VARCHAR(32)            COMMENT "FE IP that executed the statement",
  `stmt`           STRING                 COMMENT "Original SQL statement",
  `digest`         VARCHAR(32)            COMMENT "SQL fingerprint",
  `planCpuCosts`   DOUBLE                 COMMENT "CPU costs during query planning phase (nanoseconds)",
  `planMemCosts`   DOUBLE                 COMMENT "Memory costs during query planning phase (bytes)"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `clientIp`)
COMMENT "Audit log table"
PARTITION BY RANGE (`timestamp`) ()
DISTRIBUTED BY HASH (`queryId`)
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

- StarRocks v2.2.1 and subsequent minor versions:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id         VARCHAR(48)            COMMENT "Unique ID of the query",
    time             DATETIME     NOT NULL  COMMENT "Query start time",
    client_ip        VARCHAR(32)            COMMENT "Client IP",
    user             VARCHAR(64)            COMMENT "Query username",
    db               VARCHAR(96)            COMMENT "Database where the query is located",
    state            VARCHAR(8)             COMMENT "Query status (EOF, ERR, OK)",
    query_time       BIGINT                 COMMENT "Query execution time (milliseconds)",
    scan_bytes       BIGINT                 COMMENT "Bytes scanned in the query",
    scan_rows        BIGINT                 COMMENT "Record rows scanned in the query",
    return_rows      BIGINT                 COMMENT "Result rows returned in the query",
    cpu_cost_ns      BIGINT                 COMMENT "Query CPU time (nanoseconds)",
    mem_cost_bytes   BIGINT                 COMMENT "Memory consumption for the query (bytes)",
    stmt_id          INT                    COMMENT "SQL statement incremental ID",
    is_query         TINYINT                COMMENT "Whether the SQL is a query (1 or 0)",
    frontend_ip      VARCHAR(32)            COMMENT "FE IP that executed the statement",
    stmt             STRING                 COMMENT "Original SQL statement",
    digest           VARCHAR(32)            COMMENT "SQL fingerprint"
)   ENGINE = OLAP
DUPLICATE KEY (query_id, time, client_ip)
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

- StarRocks v2.2.0, v2.1.0 and subsequent minor versions:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "查询的唯一ID",
    time            DATETIME    NOT NULL  COMMENT "查询开始时间",
    client_ip       VARCHAR(32)           COMMENT "客户端IP",
    user            VARCHAR(64)           COMMENT "查询用户名",
    db              VARCHAR(96)           COMMENT "查询所在数据库",
    state           VARCHAR(8)            COMMENT "查询状态（EOF，ERR，OK）",
    query_time      BIGINT                COMMENT "查询执行时间（毫秒）",
    scan_bytes      BIGINT                COMMENT "查询扫描的字节数",
    scan_rows       BIGINT                COMMENT "查询扫描的记录行数",
    return_rows     BIGINT                COMMENT "查询返回的结果行数",
    stmt_id         INT                   COMMENT "SQL语句增量ID",
    is_query        TINYINT               COMMENT "SQL是否为查询（1或0）",
    frontend_ip     VARCHAR(32)           COMMENT "执行该语句的FE IP",
    stmt            STRING                COMMENT "原始SQL语句",
    digest          VARCHAR(32)           COMMENT "SQL指纹"
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

- StarRocks v2.0.0 及其之后小版本、v1.19.0 及其之后小版本：

```sql
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "查询的唯一ID",
    time            DATETIME    NOT NULL  COMMENT "查询开始时间",
    client_ip       VARCHAR(32)           COMMENT "客户端IP",
    user            VARCHAR(64)           COMMENT "查询用户名",
    db              VARCHAR(96)           COMMENT "查询所在数据库",
    state           VARCHAR(8)            COMMENT "查询状态（EOF，ERR，OK）",
    query_time      BIGINT                COMMENT "查询执行时间（毫秒）",
    scan_bytes      BIGINT                COMMENT "查询扫描的字节数",
    scan_rows       BIGINT                COMMENT "查询扫描的记录行数",
    return_rows     BIGINT                COMMENT "查询返回的结果行数",
    stmt_id         INT                   COMMENT "SQL语句增量ID",
    is_query        TINYINT               COMMENT "SQL是否为查询（1或0）",
    frontend_ip     VARCHAR(32)           COMMENT "执行该语句的FE IP",
    stmt            STRING                COMMENT "原始SQL语句"
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

`starrocks_audit_tbl__` 表是动态分区表。 默认情况下，第一个动态分区将在建表后 10 分钟创建。分区创建后审计日志方可导入至表中。 您可以使用以下语句检查表中的分区是否创建完成：

```sql
SHOW PARTITIONS FROM starrocks_audit_db__.starrocks_audit_tbl__;
```

待分区创建完成后，您可以继续下一步。

## 下载和配置 Audit Loader

1. [下载](https://releases.mirrorship.cn/resources/AuditLoader.zip) Audit Loader 安装包。根据 StarRocks 不同版本，该安装包包含不同的路径。您必须安装与您的 StarRocks 版本兼容的 Audit Loader 安装包。

    - **2.4**：StarRocks v2.4.0 及其之后小版本对应安装包
    - **2.3**：StarRocks v2.3.0 及其之后小版本对应安装包
    - **2.2.1+**：StarRocks v2.2.1 及其之后小版本对应安装包
    - **2.1-2.2.0**：StarRocks v2.2.0, StarRocks v2.1.0 及其之后小版本对应安装包
    - **1.18.2-2.0**：StarRocks v2.0.0 及其之后小版本、v1.19.0 及其之后小版本对应安装包

2. 解压安装包。

    ```shell
    unzip auditloader.zip
    ```

    解压生成以下文件：

    - **auditloader.jar**：插件核心代码包。
    - **plugin.properties**：插件属性文件。
    - **plugin.conf**：插件配置文件。

3. 修改 **plugin.conf** 文件以配置 Audit Loader。您必须配置以下项目以确保 Audit Loader 可以正常工作：

    - `frontend_host_port`：FE 节点 IP 地址和 HTTP 端口，格式为 `<fe_ip>:<fe_http_port>`。 默认值为 `127.0.0.1:8030`。
    - `database`：审计日志库名。
    - `table`：审计日志表名。
    - `user`：集群用户名。该用户必顇具有对应表的 INSERT 权限。
    - `password`：集群用户密码。

4. 重新打包以上文件。

    ```shell
    zip -q -m -r auditloader.zip auditloader.jar plugin.conf plugin.properties
    ```

5. 将压缩包分发至所有 FE 节点运行的机器。请确保所有压缩包都存储在相同的路径下，否则插件将安装失败。分发完成后，请复制压缩包的绝对路径。

## 安装 Audit Loader

通过以下语句安装 Audit Loader 插件：

```sql
INSTALL PLUGIN FROM "<absolute_path_to_package>";
```

详细操作说明参阅 [INSTALL PLUGIN](../sql-reference/sql-statements/Administration/INSTALL_PLUGIN.md)。

## 验证安装并查询审计日志

1. 您可以通过 [SHOW PLUGINS](../sql-reference/sql-statements/Administration/SHOW_PLUGINS.md) 语句检查插件是否安装成功。

    以下示例中，插件 `AuditLoader` 的 `Status` 为 `INSTALLED`，即代表安装成功。

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

2. 随机执行 SQL 语句以生成审计日志，并等待60秒（或您在配置 Audit Loader 时在 `max_batch_interval_sec` 项中指定的时间）以允许 Audit Loader 将审计日志攒批导入至StarRocks 中。

3. 查询审计日志表。

    ```sql
    SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__;
    ```

    以下示例演示审计日志成功导入的情况：

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

如果在动态分区创建成功且插件安装成功后仍然长时间没有审计日志导入至表中，您需要检查 **plugin.conf** 文件是否配置正确。 如需修改配置文件，您需要首先卸载插件：

```SQL
UNINSTALL PLUGIN AuditLoader;
```

所有配置设置正确后，您可以按照上述步骤重新安装 Audit Loader。