
要了解有关如何为其他对象存储创建存储卷并设置默认存储卷的更多信息，请参见[创建存储卷](../../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md)和[设置默认存储卷](../../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md)。

### 创建数据库和云原生表

创建默认存储卷后，您可以使用此存储卷创建数据库和云原生表。

共享数据StarRocks集群支持所有[StarRocks表类型](../../table_design/table_types/table_types.md)。

以下示例创建名为`cloud_db`的数据库和基于重复键表类型的`detail_demo`表，启用本地磁盘缓存，将热数据有效期设置为一个月，并禁用异步数据写入对象存储：

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
    name          CHAR(20)       NOT NULL COMMENT "范围 char(m),m 在 (1-255) ",
    profile       VARCHAR(500)   NOT NULL COMMENT "上限值为65533字节",
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
>
> 如果未指定存储卷，则在共享数据StarRocks集群中创建数据库或云原生表时将使用默认存储卷。

除了常规表`PROPERTIES`外，创建共享数据StarRocks集群表时，您需要指定以下`PROPERTIES`：

#### datacache.enable

是否启用本地磁盘缓存。

- `true`（默认值）当此属性设置为`true`时，要加载的数据同时写入对象存储和本地磁盘（作为查询加速的缓存）。
- `false`当此属性设置为`false`时，只将数据加载到对象存储中。

> **注意**
>
> 在版本3.0中，此属性称为`enable_storage_cache`。
>
> 要启用本地磁盘缓存，您必须在CN配置项`storage_root_path`中指定磁盘的目录。

#### datacache.partition_duration

热数据的有效期。启用本地磁盘缓存时，所有数据都加载到缓存中。当缓存已满时，StarRocks会从缓存中删除较早未使用的数据。当查询需要扫描已删除的数据时，StarRocks会检查数据是否位于有效期内。若数据在有效期内，则StarRocks会再次加载数据到缓存中。若数据不在有效期内，则StarRocks不会将其加载到缓存中。此属性是字符串值，可以用以下单位指定：`YEAR`、`MONTH`、`DAY`和`HOUR`，例如`7天`和`12小时`。如果未指定，所有数据都将作为热数据缓存。

> **注意**
>
> 在版本3.0中，此属性称为`storage_cache_ttl`。
>
> 只有当`datacache.enable`设置为`true`时，此属性可用。

#### enable_async_write_back

是否允许数据异步写入对象存储。默认值：`false`。
- `true`当此属性设置为`true`时，加载任务在数据写入本地磁盘缓存后立即返回成功，并且数据异步写入对象存储。这可以提高加载性能，但也会增加在潜在系统故障下的数据可靠性风险。
- `false`（默认值）当此属性设置为`false`时，加载任务仅在数据写入对象存储和本地磁盘缓存后才返回成功。这保证了更高的可用性，但会降低加载性能。

### 查看表信息

您可以使用`SHOW PROC "/dbs/<db_id>"`查看特定数据库中表的信息。有关更多信息，请参见[SHOW PROC](../../sql-reference/sql-statements/Administration/SHOW_PROC.md)。

示例：

```Plain
mysql> SHOW PROC "/dbs/xxxxx";
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| TableId | TableName   | IndexNum | PartitionColumnName | PartitionNum | State  | Type         | LastConsistencyCheckTime | ReplicaCount | PartitionType | StoragePath                  |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
| 12003   | detail_demo | 1        | NULL                | 1            | NORMAL | CLOUD_NATIVE | NULL                     | 8            | UNPARTITIONED | s3://xxxxxxxxxxxxxx/1/12003/ |
+---------+-------------+----------+---------------------+--------------+--------+--------------+--------------------------+--------------+---------------+------------------------------+
```

共享数据StarRocks集群中表的`Type`为`CLOUD_NATIVE`。在字段`StoragePath`中，StarRocks返回表存储在的对象存储目录。

### 将数据加载到共享数据StarRocks集群

共享数据StarRocks集群支持由StarRocks提供的所有加载方法。有关更多信息，请参见[数据加载概述](../../loading/Loading_intro.md)。

### 在共享数据StarRocks集群中查询

共享数据StarRocks集群中的表支持StarRocks提供的所有类型的查询。有关更多信息，请参见StarRocks [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)。

> **注意**
>
> 共享数据StarRocks集群不支持[同步物化视图](../../using_starrocks/Materialized_view-single_table.md)。