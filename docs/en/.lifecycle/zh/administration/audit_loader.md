---
displayed_sidebar: English
---

# 通过 Audit Loader 管理 StarRocks 中的审计日志

本主题介绍如何通过插件 - Audit Loader 管理 StarRocks 表中的审计日志。

StarRocks 将其审计日志存储在本地文件 **fe/log/fe.audit.log** 中，而不是内部数据库。插件 Audit Loader 允许您直接在集群中管理审计日志。Audit Loader 从文件中读取日志，并通过 HTTP PUT 将其加载到 StarRocks。

## 创建一个表来存储审计日志

在您的 StarRocks 集群中创建一个数据库和一个表来存储审计日志。有关详细说明，请参阅 [CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) 和 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

由于不同 StarRocks 版本的审计日志字段各不相同，您必须根据以下示例选择一个与您的 StarRocks 版本兼容的表结构。

> **注意**
> 不要更改示例中的表结构，否则日志加载将失败。

- StarRocks v2.4、v2.5、v3.0、v3.1 及后续小版本：

```SQL
CREATE DATABASE starrocks_audit_db__;

CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "唯一查询 ID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "查询开始时间",
  `queryType`      VARCHAR(12)            COMMENT "查询类型（query, slow_query）",
  `clientIp`       VARCHAR(32)            COMMENT "客户端 IP 地址",
  `user`           VARCHAR(64)            COMMENT "发起查询的用户",
  `authorizedUser` VARCHAR(64)            COMMENT "user_identity",
  `resourceGroup`  VARCHAR(64)            COMMENT "资源组名称",
  `catalog`        VARCHAR(32)            COMMENT "目录名称",
  `db`             VARCHAR(96)            COMMENT "查询扫描的数据库",
  `state`          VARCHAR(8)             COMMENT "查询状态（EOF, ERR, OK）",
  `errorCode`      VARCHAR(96)            COMMENT "错误代码",
  `queryTime`      BIGINT                 COMMENT "查询延迟（毫秒）",
  `scanBytes`      BIGINT                 COMMENT "扫描数据大小（字节）",
  `scanRows`       BIGINT                 COMMENT "扫描数据行数",
  `returnRows`     BIGINT                 COMMENT "结果行数",
  `cpuCostNs`      BIGINT                 COMMENT "查询 CPU 资源消耗时间（纳秒）",
  `memCostBytes`   BIGINT                 COMMENT "查询内存成本（字节）",
  `stmtId`         INT                    COMMENT "递增 SQL 语句 ID",
  `isQuery`        TINYINT                COMMENT "SQL 是否为查询（0 或 1）",
  `feIp`           VARCHAR(32)            COMMENT "执行 SQL 的 FE IP 地址",
  `stmt`           STRING                 COMMENT "SQL 语句",
  `digest`         VARCHAR(32)            COMMENT "SQL 指纹",
  `planCpuCosts`   DOUBLE                 COMMENT "规划 CPU 资源消耗时间（纳秒）",
  `planMemCosts`   DOUBLE                 COMMENT "规划内存成本（字节）"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `queryType`)
COMMENT "审计日志表"
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

- StarRocks v2.3.0 及后续小版本：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "唯一查询 ID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "查询开始时间",
  `clientIp`       VARCHAR(32)            COMMENT "客户端 IP 地址",
  `user`           VARCHAR(64)            COMMENT "发起查询的用户",
  `resourceGroup`  VARCHAR(64)            COMMENT "资源组名称",
  `db`             VARCHAR(96)            COMMENT "查询扫描的数据库",
  `state`          VARCHAR(8)             COMMENT "查询状态（EOF, ERR, OK）",
  `errorCode`      VARCHAR(96)            COMMENT "错误代码",
  `queryTime`      BIGINT                 COMMENT "查询延迟（毫秒）",
  `scanBytes`      BIGINT                 COMMENT "扫描数据大小（字节）",
  `scanRows`       BIGINT                 COMMENT "扫描数据行数",
  `returnRows`     BIGINT                 COMMENT "结果行数",
  `cpuCostNs`      BIGINT                 COMMENT "查询 CPU 资源消耗时间（纳秒）",
  `memCostBytes`   BIGINT                 COMMENT "查询内存成本（字节）",
  `stmtId`         INT                    COMMENT "递增 SQL 语句 ID",
  `isQuery`        TINYINT                COMMENT "SQL 是否为查询（0 或 1）",
  `feIp`           VARCHAR(32)            COMMENT "执行 SQL 的 FE IP 地址",
  `stmt`           STRING                 COMMENT "SQL 语句",
  `digest`         VARCHAR(32)            COMMENT "SQL 指纹",
  `planCpuCosts`   DOUBLE                 COMMENT "规划 CPU 资源消耗时间（纳秒）",
  `planMemCosts`   DOUBLE                 COMMENT "规划内存成本（字节）"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `clientIp`)
COMMENT "审计日志表"
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

- StarRocks v2.2.1 及后续小版本：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id         VARCHAR(48)            COMMENT "唯一查询 ID",
    time             DATETIME     NOT NULL  COMMENT "查询开始时间",
    client_ip        VARCHAR(32)            COMMENT "客户端 IP 地址",
    user             VARCHAR(64)            COMMENT "发起查询的用户",
    db               VARCHAR(96)            COMMENT "查询扫描的数据库",
    state            VARCHAR(8)             COMMENT "查询状态（EOF, ERR, OK）",
    query_time       BIGINT                 COMMENT "查询延迟（毫秒）",
    scan_bytes       BIGINT                 COMMENT "扫描数据大小（字节）",
    scan_rows        BIGINT                 COMMENT "扫描数据行数",
    return_rows      BIGINT                 COMMENT "结果行数",
    cpu_cost_ns      BIGINT                 COMMENT "查询 CPU 资源消耗时间（纳秒）",
    mem_cost_bytes   BIGINT                 COMMENT "查询内存成本（字节）",
    stmt_id          INT                    COMMENT "递增 SQL 语句 ID",
    is_query         TINYINT                COMMENT "SQL 是否为查询（0 或 1）",
    frontend_ip      VARCHAR(32)            COMMENT "执行 SQL 的 FE IP 地址",
    stmt             STRING                 COMMENT "SQL 语句",
    digest           VARCHAR(32)            COMMENT "SQL 指纹"
) ENGINE=OLAP
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

- StarRocks v2.2.0、v2.1.0 及后续小版本：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "唯一查询 ID",
    time            DATETIME    NOT NULL  COMMENT "查询开始时间",
    client_ip       VARCHAR(32)           COMMENT "客户端 IP 地址",
    user            VARCHAR(64)           COMMENT "发起查询的用户",
    db              VARCHAR(96)           COMMENT "查询扫描的数据库",
    state           VARCHAR(8)            COMMENT "查询状态（EOFE, RR, OK）",
    query_time      BIGINT                COMMENT "查询延迟（毫秒）",
    scan_bytes      BIGINT                COMMENT "扫描数据大小（字节）",
    scan_rows       BIGINT                COMMENT "扫描数据行数",
```
如果在创建动态分区并安装插件后，表中没有加载审计日志，您可以检查**plugin.conf**是否配置正确。要修改它，您必须先卸载插件：

```SQL
UNINSTALL PLUGIN AuditLoader;
```

在所有配置正确设置后，您可以按照上述步骤再次安装 Audit Loader。
```SQL
如果在创建动态分区并安装插件后，审计日志未加载到表中，您可以检查**plugin.conf**是否配置正确。要修改它，您必须先卸载插件：

UNINSTALL PLUGIN AuditLoader;

在所有配置正确设置后，您可以按照上述步骤重新安装 Audit Loader。