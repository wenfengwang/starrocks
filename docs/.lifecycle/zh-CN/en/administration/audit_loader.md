---
displayed_sidebar: "Chinese"
---

# 通过Audit Loader管理StarRocks中的审计日志

本主题描述如何通过Audit Loader插件在表内管理StarRocks审计日志。

StarRocks将其审计日志存储在本地文件**fe/log/fe.audit.log**中，而不是内部数据库。Audit Loader插件允许您直接在集群内管理审计日志。Audit Loader从文件中读取日志，并通过HTTP PUT将其加载到StarRocks中。

## 创建用于存储审计日志的表

在您的StarRocks集群中创建数据库和表以存储其审计日志。有关详细说明，请参阅[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)和[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

由于不同StarRocks版本的审计日志字段不同，您必须从以下示例中选择一个来创建与您的StarRocks兼容的表。

> **注意**
>
> 请勿更改示例中的表模式，否则日志加载将失败。

- StarRocks v2.4、v2.5、v3.0、v3.1和之后的小版本：

```SQL
CREATE DATABASE starrocks_audit_db__;

CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "唯一查询ID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "查询开始时间",
  `queryType`      VARCHAR(12)            COMMENT "查询类型（query, slow_query）",
  `clientIp`       VARCHAR(32)            COMMENT "客户端IP地址",
  `user`           VARCHAR(64)            COMMENT "发起查询的用户",
  `authorizedUser` VARCHAR(64)            COMMENT "授权用户标识",
  `resourceGroup`  VARCHAR(64)            COMMENT "资源组名称",
  `catalog`        VARCHAR(32)            COMMENT "数据库名",
  `db`             VARCHAR(96)            COMMENT "查询扫描的数据库",
  `state`          VARCHAR(8)             COMMENT "查询状态（EOF, ERR, OK）",
  `errorCode`      VARCHAR(96)            COMMENT "错误码",
  `queryTime`      BIGINT                 COMMENT "查询延迟时间（毫秒）",
  `scanBytes`      BIGINT                 COMMENT "扫描数据大小（字节）",
  `scanRows`       BIGINT                 COMMENT "扫描数据行数",
  `returnRows`     BIGINT                 COMMENT "结果行数",
  `cpuCostNs`      BIGINT                 COMMENT "查询消耗的CPU资源时间（纳秒）",
  `memCostBytes`   BIGINT                 COMMENT "查询消耗的内存（字节）",
  `stmtId`         INT                    COMMENT "增量SQL语句ID",
  `isQuery`        TINYINT                COMMENT "SQL是否为查询（0和1）",
  `feIp`           VARCHAR(32)            COMMENT "执行SQL的FE的IP地址",
  `stmt`           STRING                 COMMENT "SQL语句",
  `digest`         VARCHAR(32)            COMMENT "SQL指纹",
  `planCpuCosts`   DOUBLE                 COMMENT "规划消耗的CPU资源时间（纳秒）",
  `planMemCosts`   DOUBLE                 COMMENT "规划消耗的内存（字节）"
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

- StarRocks v2.3.0和之后的小版本：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "唯一查询ID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "查询开始时间",
  `clientIp`       VARCHAR(32)            COMMENT "客户端IP地址",
  `user`           VARCHAR(64)            COMMENT "发起查询的用户",
  `resourceGroup`  VARCHAR(64)            COMMENT "资源组名称",
  `db`             VARCHAR(96)            COMMENT "查询扫描的数据库",
  `state`          VARCHAR(8)             COMMENT "查询状态（EOF, ERR, OK）",
  `errorCode`      VARCHAR(96)            COMMENT "错误码",
  `queryTime`      BIGINT                 COMMENT "查询延迟时间（毫秒）",
  `scanBytes`      BIGINT                 COMMENT "扫描数据大小（字节）",
  `scanRows`       BIGINT                 COMMENT "扫描数据行数",
  `returnRows`     BIGINT                 COMMENT "结果行数",
  `cpuCostNs`      BIGINT                 COMMENT "查询消耗的CPU资源时间（纳秒）",
  `memCostBytes`   BIGINT                 COMMENT "查询消耗的内存（字节）",
  `stmtId`         INT                    COMMENT "增量SQL语句ID",
  `isQuery`        TINYINT                COMMENT "SQL是否为查询（0和1）",
  `feIp`           VARCHAR(32)            COMMENT "执行SQL的FE的IP地址",
  `stmt`           STRING                 COMMENT "SQL语句",
  `digest`         VARCHAR(32)            COMMENT "SQL指纹"
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

- StarRocks v2.2.1和之后的小版本：

```SQL

CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id         VARCHAR(48)            COMMENT "唯一查询ID",
    time             DATETIME     NOT NULL  COMMENT "查询开始时间",
    client_ip        VARCHAR(32)            COMMENT "客户端IP地址",
    user             VARCHAR(64)            COMMENT "发起查询的用户",
    db               VARCHAR(96)            COMMENT "查询扫描的数据库",
    state            VARCHAR(8)             COMMENT "查询状态（EOF, ERR, OK）",
    query_time       BIGINT                 COMMENT "查询延迟时间（毫秒）",
    scan_bytes       BIGINT                 COMMENT "扫描数据大小（字节）",
    scan_rows        BIGINT                 COMMENT "扫描数据行数",
    return_rows      BIGINT                 COMMENT "结果行数",
    cpu_cost_ns      BIGINT                 COMMENT "查询消耗的CPU资源时间（纳秒）",
    mem_cost_bytes   BIGINT                 COMMENT "查询消耗的内存（字节）",
    stmt_id          INT                    COMMENT "增量SQL语句ID",
    is_query         TINYINT                COMMENT "SQL是否为查询（0和1）",
    frontend_ip      VARCHAR(32)            COMMENT "执行SQL的前端IP地址",
    stmt             STRING                 COMMENT "SQL语句",
    digest           VARCHAR(32)            COMMENT "SQL指纹"
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


- StarRocks v2.2.0、v2.1.0和之后的小版本：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "唯一查询ID",
    time            DATETIME    NOT NULL  COMMENT "查询开始时间",
    client_ip       VARCHAR(32)           COMMENT "客户端IP地址",
    user            VARCHAR(64)           COMMENT "发起查询的用户",
    db              VARCHAR(96)           COMMENT "查询扫描的数据库",
    state           VARCHAR(8)            COMMENT "查询状态（EOFE, RR, OK）",
    query_time      BIGINT                COMMENT "查询延迟时间（毫秒）",
    scan_bytes      BIGINT                COMMENT "扫描数据大小（字节）",
    scan_rows       BIGINT                COMMENT "扫描数据行数",
    return_rows     BIGINT                COMMENT "结果行数",
    stmt_id         INT                   COMMENT "增量SQL语句ID",
```markdown
- 如果 SQL 是查询（0 和 1）

    frontend_ip VARCHAR（32）备注 "执行 SQL 的 FE 的 IP 地址"

    stmt STRING备注“SQL 语句”

    摘要 VARCHAR（32）备注“SQL 指纹”
）engine=OLAP
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

- StarRocks v2.0.0及更高版本的次要版本，StarRocks v1.19.0及更高版本的次要版本：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id VARCHAR(48)备注“唯一查询ID”
    时间 DATETIME NOT NULL备注“查询开始时间”
    client_ip VARCHAR(32)备注“客户端 IP 地址”
    user VARCHAR(64)备注“发起查询的用户”
    db VARCHAR(96)备注“查询扫描的数据库”
    state VARCHAR(8)备注“查询状态（EOF、ERR、OK）”
    query_time BIGINT备注“查询延迟时间（毫秒）”
    scan_bytes BIGINT备注“扫描数据的大小（字节）”
    scan_rows BIGINT备注“扫描数据的行数”
    return_rows BIGINT备注“结果的行数”
    stmt_id INT备注“递增的 SQL 语句ID”
    is_query TINYINT备注“If the SQL is a query（0 and 1）”
    frontend_ip VARCHAR(32)备注“执行 SQL 的 FE 的 IP 地址”
    stmt STRING备注“SQL 语句”
）engine=OLAP
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

`starrocks_audit_tbl__`带有动态分区创建。默认情况下，表创建后10分钟后创建第一个动态分区。然后可以将审计日志加载到表中。您可以使用以下语句检查表中的分区：

```SQL
SHOW PARTITIONS FROM starrocks_audit_db__.starrocks_audit_tbl__;
```

创建分区后，您可以进行下一步。

## 下载和配置 Audit Loader

1. [下载](https://releases.starrocks.io/resources/AuditLoader.zip) Audit Loader 安装程序包。该程序包包含多个不同 StarRocks 版本的目录。您必须导航到相应的目录并安装与您的 StarRocks 兼容的程序包。

    - **2.4**：StarRocks v2.4.0及更高版本的次要版本
    - **2.3**：StarRocks v2.3.0及更高版本的次要版本
    - **2.2.1+**：StarRocks v2.2.1及更高版本的次要版本
    - **2.1-2.2.0**：StarRocks v2.2.0、StarRocks v2.1.0及更高版本的次要版本
    - **1.18.2-2.0**：StarRocks v2.0.0及更高版本的次要版本, StarRocks v1.19.0及更高版本的次要版本

2. 解压安装程序包。

    ```shell
    unzip auditloader.zip
    ```

    以下文件被解压：

    - **auditloader.jar**：Audit Loader 的 JAR 文件。
    - **plugin.properties**：Audit Loader 的属性文件。
    - **plugin.conf**：Audit Loader 的配置文件。

3. 修改 **plugin.conf** 以配置 Audit Loader。您必须配置以下内容以确保 Audit Loader 能够正常工作：

    - `frontend_host_port`：FE IP 地址和 HTTP 端口，格式为 `<fe_ip>:<fe_http_port>`。默认值为 `127.0.0.1:8030`。
    - `database`：您创建用于托管审计日志的数据库的名称。
    - `table`：您创建用于托管审计日志的表的名称。
    - `user`：您的群集用户名。您必须具有加载数据（LOAD_PRIV）到表中的权限。
    - `password`：您的用户密码。

4. 将文件重新打包为一个程序包。

    ```shell
    zip -q -m -r auditloader.zip auditloader.jar plugin.conf plugin.properties
    ```

5. 将程序包分发到所有托管 FE 节点的机器上。确保所有程序包存储在相同的路径中。否则，安装将失败。记得在分发程序包后复制绝对路径。

## 安装 Audit Loader

执行以下语句并附带您复制的路径将 Audit Loader 安装为 StarRocks 中的插件：

```SQL
INSTALL PLUGIN FROM "<absolute_path_to_package>";
```

请参阅 [INSTALL PLUGIN](../sql-reference/sql-statements/Administration/INSTALL_PLUGIN.md) 以获取详细说明。

## 验证安装并查询审计日志

1. 您可以通过 [SHOW PLUGINS](../sql-reference/sql-statements/Administration/SHOW_PLUGINS.md) 检查安装是否成功。

    在下面的示例中，插件 `AuditLoader` 的 `Status` 为 `INSTALLED`，表示安装成功。

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

2. 执行一些随机的 SQL 以生成审计日志，并等待 60 秒（或者您在配置 Audit Loader 时指定的 `max_batch_interval_sec` 时间）以允许 Audit Loader 将审计日志加载到 StarRocks 中。

3. 通过查询表来检查审计日志。

    ```SQL
    SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__;
    ```

    以下示例显示了审计日志何时成功加载到表中：

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

如果创建动态分区后插件安装后未加载任何审计日志，您可以检查 **plugin.conf** 是否正确配置。要进行修改，必须首先卸载插件：

```SQL
UNINSTALL PLUGIN AuditLoader;
```

在所有配置正确后，您可以按照上述步骤再次安装 Audit Loader。
```