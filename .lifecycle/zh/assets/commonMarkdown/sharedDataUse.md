
有关为其他对象存储创建存储卷及设置默认存储卷的更多信息，请参考[CREATE STORAGE VOLUME](../../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md)和[SET DEFAULT STORAGE VOLUME](../../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md)。

### 创建数据库和云原生表

在创建默认存储卷之后，您可以利用这个存储卷来创建数据库和云原生表。

共享数据的StarRocks集群支持所有[StarRocks表类型](../../table_design/table_types/table_types.md)。

以下示例展示了如何基于Duplicate Key表类型创建一个名为cloud_db的数据库和一个名为detail_demo的表，启用本地磁盘缓存，将热数据的有效期设置为一个月，并关闭对象存储的异步数据摄取功能：

```SQL
CREATE DATABASE cloud_db;
USE cloud_db;
CREATE TABLE IF NOT EXISTS detail_demo (
    recruit_date  DATE           NOT NULL COMMENT "YYYY-MM-DD",
    region_num    TINYINT        COMMENT "range [-128, 127]",
    num_plate     SMALLINT       COMMENT "range [-32768, 32767] ",
    tel           INT            COMMENT "range [-2147483648, 2147483647]",
    id            BIGINT         COMMENT "range [-2^63 + 1 ~ 2^63 - 1]",
    password      LARGEINT       COMMENT "range [-2^127 + 1 ~ 2^127 - 1]",
    name          CHAR(20)       NOT NULL COMMENT "range char(m),m in (1-255) ",
    profile       VARCHAR(500)   NOT NULL COMMENT "upper limit value 65533 bytes",
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
> 如果没有指定存储卷，在共享数据的**StarRocks**集群中创建数据库或云原生表时，默认会使用默认存储卷。

在为共享数据的StarRocks集群创建表时，除了常规表属性（PROPERTIES）外，您还需要指定以下属性：

#### datacache.enable

是否启用本地磁盘缓存。

- true（默认值）当此属性被设置为true时，待加载的数据会同时写入对象存储和本地磁盘（作为查询加速的缓存）。
- false 当此属性被设置为false时，数据仅加载到对象存储中。

> **注意**
> 在3.0版本中，此属性的名称为`enable_storage_cache`。
> 要启用本地磁盘缓存，您必须在CN配置项 `storage_root_path` 中指定磁盘目录。

#### datacache.partition_duration

热数据的有效期。启用本地磁盘缓存后，所有数据都会被加载到缓存中。当缓存满了，StarRocks会删除最近使用较少的数据。如果查询需要扫描已删除的数据，StarRocks会检查该数据是否在有效期内。如果数据在有效期内，StarRocks会重新将其加载到缓存中；如果不在，StarRocks不会加载到缓存。此属性是一个字符串值，可以用YEAR、MONTH、DAY和HOUR等单位来指定，例如7 DAY或12 HOUR。如果不进行指定，则所有数据都会被视为热数据并被缓存。

> **注意**
> 在3.0版本中，此属性的名称为 `storage_cache_ttl`。
> 只有当 `datacache.enable` 被设置为 `true` 时，此属性才可用。

#### enable_async_write_back

是否允许数据异步写入对象存储。默认值为false。
- true 当此属性被设置为true时，一旦数据被写入本地磁盘缓存，加载任务就会返回成功，并且数据会被异步写入对象存储。这样可以提高加载性能，但在系统可能出现故障时，也可能会影响数据的可靠性。
- false（默认值）当此属性被设置为false时，只有在数据同时被写入对象存储和本地磁盘缓存后，加载任务才会返回成功。这样可以确保更高的可用性，但加载性能可能会有所下降。

### 查看表信息

您可以通过使用 `SHOW PROC "/dbs/
<db_id>
"` 来查看特定数据库中的表信息。更多信息请参考[SHOW PROC](../../sql-reference/sql-statements/Administration/SHOW_PROC.md)。

示例：

```Plain
mysql> SHOW PROC "/dbs/xxxxx";
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| TableId | TableName   | IndexNum | PartitionColumnName | PartitionNum | State  | Type         | LastConsistencyCheckTime | ReplicaCount | PartitionType | StoragePath                  |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| 12003   | detail_demo | 1        | NULL                | 1            | NORMAL | CLOUD_NATIVE | NULL                     | 8            | UNPARTITIONED | s3://xxxxxxxxxxxxxx/1/12003/ |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
```

在共享数据的StarRocks集群中，表的类型为CLOUD_NATIVE。在StoragePath字段中，StarRocks会返回存储该表的对象存储路径。

### 将数据加载到共享数据的StarRocks集群

共享数据的StarRocks集群支持所有加载方法提供的StarRocks。请参阅[数据加载概述](../../loading/Loading_intro.md)了解更多信息。

### 在共享数据的StarRocks集群中进行查询

共享数据的StarRocks集群中的表支持StarRocks提供的所有查询类型。更多信息请参考StarRocks [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)。

> **注意**
> 共享数据的StarRocks集群不支持[synchronous materialized views](../../using_starrocks/Materialized_view-single_table.md)。
