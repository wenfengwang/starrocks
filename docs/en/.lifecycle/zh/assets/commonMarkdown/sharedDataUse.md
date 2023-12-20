
有关如何为其他对象存储创建存储卷并设置默认存储卷的更多信息，请参阅 [CREATE STORAGE VOLUME](../../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md) 和 [SET DEFAULT STORAGE VOLUME](../../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md)。

### 创建数据库和云原生表

创建默认存储卷后，您可以使用该存储卷创建数据库和云原生表。

Shared-data StarRocks 集群支持所有 [StarRocks 表类型](../../table_design/table_types/table_types.md)。

以下示例基于 Duplicate Key 表类型创建数据库 `cloud_db` 和表 `detail_demo`，启用本地磁盘缓存，设置热数据有效期为一个月，并禁用异步数据写入到对象存储：

```SQL
CREATE DATABASE cloud_db;
USE cloud_db;
CREATE TABLE IF NOT EXISTS detail_demo (
    recruit_date  DATE           NOT NULL COMMENT "YYYY-MM-DD",
    region_num    TINYINT        COMMENT "范围 [-128, 127]",
    num_plate     SMALLINT       COMMENT "范围 [-32768, 32767] ",
    tel           INT            COMMENT "范围 [-2147483648, 2147483647]",
    id            BIGINT         COMMENT "范围 [-2^63 + 1 ~ 2^63 - 1]",
    password      LARGEINT       COMMENT "范围 [-2^127 + 1 ~ 2^127 - 1]",
    name          CHAR(20)       NOT NULL COMMENT "范围 char(m), m 在 (1-255) ",
    profile       VARCHAR(500)   NOT NULL COMMENT "上限值 65533 字节",
    ispass        BOOLEAN        COMMENT "true/false")
DUPLICATE KEY(recruit_date, region_num)
DISTRIBUTED BY HASH(recruit_date, region_num)
PROPERTIES (
    "storage_volume" = "def_volume",
    "datacache.enable" = "true",
    "datacache.partition_duration" = "1 MONTH",
    "enable_async_write_back" = "false"
);
```

> **注意**
> 如果未指定存储卷，则在共享数据 StarRocks 集群中创建数据库或云原生表时将使用默认存储卷。

除了常规表的 PROPERTIES 之外，在为共享数据 StarRocks 集群创建表时还需要指定以下 PROPERTIES：

#### datacache.enable

是否启用本地磁盘缓存。

- `true`（默认）当此属性设置为 `true` 时，待加载的数据会同时写入对象存储和本地磁盘（作为查询加速的缓存）。
- `false` 当此属性设置为 `false` 时，数据仅加载到对象存储中。

> **注意**
> 在版本 3.0 中，此属性被命名为 `enable_storage_cache`。
> 要启用本地磁盘缓存，必须在 CN 配置项 `storage_root_path` 中指定磁盘的目录。

#### datacache.partition_duration

热数据的有效期。当启用本地磁盘缓存时，所有数据都会加载到缓存中。当缓存满时，StarRocks 会从缓存中删除最近较少使用的数据。当查询需要扫描已删除的数据时，StarRocks 会检查数据是否在有效期内。如果数据在有效期内，StarRocks 会再次将数据加载到缓存中。如果数据不在有效期内，StarRocks 不会将其加载到缓存中。此属性是一个字符串值，可以使用以下单位指定：`YEAR`、`MONTH`、`DAY` 和 `HOUR`，例如 `7 DAY` 和 `12 HOUR`。如果不指定，则所有数据都作为热数据进行缓存。

> **注意**
> 在版本 3.0 中，此属性被命名为 `storage_cache_ttl`。
> 仅当 `datacache.enable` 设置为 `true` 时，此属性才可用。

#### enable_async_write_back

是否允许数据异步写入对象存储。默认值：`false`。
- `true` 当此属性设置为 `true` 时，加载任务一旦将数据写入本地磁盘缓存就返回成功，并且数据会被异步写入对象存储。这可以提高加载性能，但在潜在的系统故障下也可能会带来数据可靠性的风险。
- `false`（默认）当此属性设置为 `false` 时，只有在数据同时写入对象存储和本地磁盘缓存后，加载任务才会返回成功。这确保了更高的可用性，但可能会降低加载性能。

### 查看表信息

您可以使用 `SHOW PROC "/dbs/<db_id>"` 查看特定数据库中表的信息。有关详细信息，请参阅 [SHOW PROC](../../sql-reference/sql-statements/Administration/SHOW_PROC.md)。

示例：

```Plain
mysql> SHOW PROC "/dbs/xxxxx";
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| TableId | TableName   | IndexNum | PartitionColumnName | PartitionNum | State  | Type         | LastConsistencyCheckTime | ReplicaCount | PartitionType | StoragePath                  |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| 12003   | detail_demo | 1        | NULL                | 1            | NORMAL | CLOUD_NATIVE | NULL                     | 8            | UNPARTITIONED | s3://xxxxxxxxxxxxxx/1/12003/ |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
```

共享数据 StarRocks 集群中表的 `Type` 为 `CLOUD_NATIVE`。在 `StoragePath` 字段中，StarRocks 返回存储表的对象存储目录。

### 将数据加载到共享数据 StarRocks 集群中

共享数据 StarRocks 集群支持 StarRocks 提供的所有数据加载方法。有关更多信息，请参阅 [数据加载概述](../../loading/Loading_intro.md)。

### 在共享数据 StarRocks 集群中查询

共享数据 StarRocks 集群中的表支持 StarRocks 提供的所有查询类型。有关更多信息，请参阅 StarRocks [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)。

> **注意**
> 共享数据 StarRocks 集群不支持 [同步物化视图](../../using_starrocks/Materialized_view-single_table.md)。