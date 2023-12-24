---
displayed_sidebar: English
---

# 表达式分区（推荐）

从 v3.0 开始，StarRocks 支持表达式分区（以前称为自动分区），这种分区方式更加灵活和用户友好。适用于大多数场景，例如基于连续时间范围或枚举值查询和管理数据。

您只需在创建表时指定一个简单的分区表达式（时间函数表达式或列表达式）。在数据加载过程中，StarRocks 会根据数据和分区表达式中定义的规则自动创建分区。您不再需要在创建表时手动创建大量分区，也不再需要配置动态分区属性。

## 基于时间函数表达式的分区

如果您经常基于连续时间范围查询和管理数据，则只需在时间函数表达式中指定日期类型（DATE或DATETIME）列作为分区列，并指定年、月、日、小时作为分区粒度。StarRocks 会自动创建分区，并根据加载的数据和分区表达式设置分区的开始和结束日期或日期时间。

但是，在某些特殊场景下，例如将历史数据按月划分为分区，将最近数据按天划分为分区，则必须使用[范围分区](./Data_distribution.md#range-partitioning)来创建分区。

### 语法

```sql
PARTITION BY expression
...
[ PROPERTIES( 'partition_live_number' = 'xxx' ) ]

expression ::=
    { date_trunc ( <time_unit> , <partition_column> ) |
      time_slice ( <partition_column> , INTERVAL <N> <time_unit> [ , boundary ] ) }
```

### 参数

| 参数              | 必填 | 描述                                                  |
| ----------------------- | -------- | ------------------------------------------------------------ |
| `expression`            |     是     | 目前仅支持 [date_trunc](../sql-reference/sql-functions/date-time-functions/date_trunc.md) 和 [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md) 函数。如果使用 `time_slice` 函数，则不需要传递 `boundary` 参数。这是因为在此方案中，该参数的默认值和有效值是 `floor`，而该值不能是 `ceil`。 |
| `time_unit`             |       是   | 分区粒度，可以是 `hour`、 `day`、 `month` 或 `year`。不支持 `week` 分区粒度。如果分区粒度为 `hour`，则分区列必须为 DATETIME 数据类型，不能为 DATE 数据类型。 |
| `partition_column` |     是     | 分区列的名称。<br/><ul><li>分区列只能是 DATE 或 DATETIME 数据类型。分区列允许 `NULL` 值。</li><li>如果使用 `date_trunc` 函数，则分区列可以是 DATE 或 DATETIME 数据类型。如果使用 `time_slice` 函数，则分区列必须是 DATETIME 数据类型。 </li><li>如果分区列的数据类型为 DATE，则支持的范围为 [0000-01-01 ~ 9999-12-31]。如果分区列的数据类型为 DATETIME，则支持的范围为 [0000-01-01 01:01:01 ~ 9999-12-31 23:59:59]。</li><li>目前只能指定一个分区列，不支持多个分区列。</li></ul> |
| `partition_live_number` |      否    | 要保留的最新分区数。“最近”是指分区按时间顺序排序，**以当前日期为基准**，保留倒数的分区数，删除其余分区（更早创建的分区）。StarRocks 调度任务来管理分区数量，调度间隔可以通过 FE 动态参数进行配置，`dynamic_partition_check_interval_seconds` 默认为 600 秒（10 分钟）。假设当前日期为 2023 年 4 月 4 日，`partition_live_number` 设置为 `2`，分区包括 `p20230401`、 `p20230402` 、  `p20230403`、 `p20230404`。分区 `p20230403` 和 `p20230404` 将被保留，其他分区将被删除。如果加载了脏数据，例如未来日期 4 月 5 日和 4 月 6 日的数据，则分区包括  、 `p20230401`  、 `p20230402` `p20230403`、 `p20230404` `p20230405`和 `p20230406`。然后 `p20230403` 保留分区 、 `p20230404`、 `p20230405`和 ， `p20230406` 删除其他分区。 |

### 使用说明

- 在数据加载过程中，StarRocks 会根据加载的数据自动创建一些分区，但如果加载作业因某种原因失败，StarRocks 自动创建的分区将无法自动删除。
- StarRocks 默认自动创建的分区数量上限为 4096，可通过 FE 参数进行配置 `max_automatic_partition_number`。此参数可以防止意外创建过多的分区。
- 分区的命名规则与动态分区的命名规则一致。

### **例子**

示例 1：假设您每天频繁查询数据。您可以使用分区表达式 `date_trunc()` ，并将分区列设置为 `event_day` 在创建表时设置分区粒度 `day` 。在加载过程中，数据会根据日期自动分区。当天的数据存储在一个分区中，分区修剪可以显著提高查询效率。

```SQL
CREATE TABLE site_access1 (
    event_day DATETIME NOT NULL,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY date_trunc('day', event_day)
DISTRIBUTED BY HASH(event_day, site_id);
```

例如，当加载以下两个数据行时，StarRocks 会自动创建两个分区，`p20230226` 范围 `p20230227` 分别为 [2023-02-26 00：00：00， 2023-02-27 00：00：00] 和 [2023-02-27 00：00：00， 2023-02-28 00：00：00）。如果后续加载的数据在这些范围内，它们会自动路由到相应的分区。

```SQL
-- insert two data rows
INSERT INTO site_access1  
    VALUES ("2023-02-26 20:12:04",002,"New York","Sam Smith",1),
           ("2023-02-27 21:06:54",001,"Los Angeles","Taylor Swift",1);

-- view partitions
mysql > SHOW PARTITIONS FROM site_access1;
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range                                                                                                | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 17138       | p20230226     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-26 00:00:00]; ..types: [DATETIME]; keys: [2023-02-27 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
| 17113       | p20230227     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-27 00:00:00]; ..types: [DATETIME]; keys: [2023-02-28 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
2 rows in set (0.00 sec)
```

示例2：如果要实现分区生命周期管理，即只保留一定数量的最近分区，删除历史分区，可以使用 `partition_live_number` 属性指定要保留的分区数。

```SQL
CREATE TABLE site_access2 (
    event_day DATETIME NOT NULL,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
) 
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY date_trunc('month', event_day)
DISTRIBUTED BY HASH(event_day, site_id)
PROPERTIES(
    "partition_live_number" = "3" -- 仅保留最近的三个分区
);
```

示例 3：假设您经常按周查询数据。您可以使用分区表达式，并在创建表时将分区列设置为 `event_day`，使用 `time_slice()` 将分区粒度设置为 7 天。一周的数据存储在一个分区中，分区修剪可以显著提高查询效率。

```SQL
CREATE TABLE site_access(
    event_day DATETIME NOT NULL,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY time_slice(event_day, INTERVAL 7 day)
DISTRIBUTED BY HASH(event_day, site_id)
```

## 基于列表达式的分区（从 v3.1 开始）


如果您经常查询和管理特定类型的数据，则只需指定表示该类型的列作为分区列。StarRocks 会根据加载的数据的分区列值自动创建分区。

但是，在某些特殊场景下，例如当表中包含列 `city` 时，您经常根据国家和城市查询和管理数据。您必须使用[列表分区](./list_partitioning.md)来将同一国家/地区内多个城市的数据存储在一个分区中。

### 语法

```sql
PARTITION BY expression
...
[ PROPERTIES("partition_live_number" = "xxx") ]

expression ::=
    ( partition_columns )
    
partition_columns ::=
    <column>, [ <column> [,...] ]
```

### 参数

| **参数**          | **必填** | **描述**                                                  |
| ----------------------- | -------- | ------------------------------------------------------------ |
| `partition_columns`     | 是      | 分区列的名称。<br/> <ul><li>分区列值可以是字符串（不支持 BINARY）、日期或日期时间、整数和布尔值。分区列允许 `NULL` 值。</li><li> 每个分区只能包含分区列具有相同值的数据。若要在分区的分区列中包含具有不同值的数据，请参阅 [列表分区](./list_partitioning.md)。</li></ul> |
| `partition_live_number` | 否      | 要保留的分区数。比较分区之间的分区列值，定期删除值较小的分区，同时保留值较大的分区。<br/>StarRocks 调度任务来管理分区数量，调度间隔可以通过 FE 动态参数进行配置， `dynamic_partition_check_interval_seconds` 默认为 600 秒（10 分钟）。<br/>**注意**<br/>如果分区列中的值为字符串，StarRocks 会比较分区名称的字典顺序，并定期保留较早出现的分区，同时删除较晚的分区。 |

### 使用说明

- 在数据加载过程中，StarRocks 会根据加载的数据自动创建一些分区，但如果加载作业因某种原因失败，StarRocks 自动创建的分区将无法自动删除。
- StarRocks 默认自动创建的分区数量上限为 4096，可通过 FE 参数进行配置 `max_automatic_partition_number`。此参数可以防止意外创建过多的分区。
- 分区的命名规则：如果指定了多个分区列，则不同分区列的值在分区名称中用下划线连接， `_` 格式为 `p<value in partition column 1>_<value in partition column 2>_...`. 例如，如果将两个 `dt` `province` 列和指定为分区列（两者都是字符串类型），并且加载了一个带有值 和 的数据行 `2022-04-01` `beijing` ，则自动创建的相应分区将命名为 `p20220401_beijing`。

### 例子

示例1：假设您经常根据时间范围和具体城市查询数据中心计费明细。在创建表时，可以使用分区表达式将第一个分区列指定为 `dt`  和 `city` 。这样，属于同一日期和城市的数据被路由到同一个分区中，分区修剪可以显著提高查询效率。

```SQL
CREATE TABLE t_recharge_detail1 (
    id bigint,
    user_id bigint,
    recharge_money decimal(32,2), 
    city varchar(20) not null,
    dt varchar(20) not null
)
DUPLICATE KEY(id)
PARTITION BY (dt,city)
DISTRIBUTED BY HASH(`id`);
```

在表中插入单个数据行。

```SQL
INSERT INTO t_recharge_detail1 
    VALUES (1, 1, 1, 'Houston', '2022-04-01');
```

查看分区。结果显示，StarRocks 会根据加载的数据自动创建分区 `p20220401_Houston1`。在随后的加载过程中，带有值 `2022-04-01` 和 `Houston` 的数据在分区列中 `dt` `city` 存储，并存储在该分区中。

> **注意**
>
> 每个分区只能包含具有分区列指定一个值的数据。若要为分区中的分区列指定多个值，请参阅 [列表分区](./list_partitioning.md)。

```SQL
MySQL > SHOW PARTITIONS from t_recharge_detail1\G
*************************** 1. row ***************************
             PartitionId: 16890
           PartitionName: p20220401_Houston
          VisibleVersion: 2
      VisibleVersionTime: 2023-07-19 17:24:53
      VisibleVersionHash: 0
                   State: NORMAL
            PartitionKey: dt, city
                    List: (('2022-04-01', 'Houston'))
         DistributionKey: id
                 Buckets: 6
          ReplicationNum: 3
           StorageMedium: HDD
            CooldownTime: 9999-12-31 23:59:59
LastConsistencyCheckTime: NULL
                DataSize: 2.5KB
              IsInMemory: false
                RowCount: 1
1 row in set (0.00 sec)
```

示例 2：您还可以在创建表时配置 “`partition_live_number`” 属性进行分区生命周期管理，例如指定该表只保留 3 个分区。

```SQL
CREATE TABLE t_recharge_detail2 (
    id bigint,
    user_id bigint,
    recharge_money decimal(32,2), 
    city varchar(20) not null,
    dt varchar(20) not null
)
DUPLICATE KEY(id)
PARTITION BY (dt,city)
DISTRIBUTED BY HASH(`id`) 
PROPERTIES(
    "partition_live_number" = "3" -- only retains the most recent three partitions
);
```

## 管理分区

### 将数据加载到分区中

在数据加载过程中，StarRocks 会根据加载的数据和分区表达式定义的分区规则自动创建分区。

请注意，如果在创建表时使用表达式分区，并且需要使用 [INSERT OVERWRITE](../loading/InsertInto.md#overwrite-data-via-insert-overwrite-select) 覆盖特定分区中的数据，则无论该分区是否已创建，您当前都需要在 中显式提供分区范围`PARTITION()`。这与[范围分区](./Data_distribution.md#range-partitioning)或[列表分区](./list_partitioning.md)不同，后者只允许您在 中提供分区名称`PARTITION (<partition_name>)`。

如果您在创建表时使用时间函数表达式，并希望覆盖特定分区中的数据，则需要提供该分区的开始日期或日期时间（创建表时配置的分区粒度）。如果分区不存在，可以在数据加载时自动创建。

```SQL
INSERT OVERWRITE site_access1 PARTITION(event_day='2022-06-08 20:12:04')
    SELECT * FROM site_access2 PARTITION(p20220608);
```

如果在创建表时使用列表达式，并且想要覆盖特定分区中的数据，则需要提供分区包含的分区列值。如果分区不存在，可以在数据加载时自动创建。

```SQL
INSERT OVERWRITE t_recharge_detail1 PARTITION(dt='2022-04-02',city='texas')
    SELECT * FROM t_recharge_detail2 PARTITION(p20220402_texas);
```

### 查看分区

当您要查看有关自动创建的分区的特定信息时，您需要使用该 `SHOW PARTITIONS FROM <table_name>` 语句。该 `SHOW CREATE TABLE <table_name>` 语句仅返回在创建表时配置的表达式分区语法。

```SQL
MySQL > SHOW PARTITIONS FROM t_recharge_detail1;
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName     | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | List                        | DistributionKey | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 16890       | p20220401_Houston | 2              | 2023-07-19 17:24:53 | 0                  | NORMAL | dt, city     | (('2022-04-01', 'Houston')) | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
| 17056       | p20220402_texas   | 2              | 2023-07-19 17:27:42 | 0                  | NORMAL | dt, city     | (('2022-04-02', 'texas'))   | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
2 rows in set (0.00 sec)
```

## 限制

- 从 v3.1.0 开始，StarRocks 的[共享数据模式](../deployment/shared_data/s3.md)支持[时间函数表达式](#partitioning-based-on-a-time-function-expression)。从 v3.1.1 开始，StarRocks 的[共享数据模式](../deployment/shared_data/s3.md)进一步支持[列表表达式](#partitioning-based-on-the-column-expression-since-v31)。
- 目前，不支持使用 CTAS 创建表配置表达式分区。
- 目前不支持使用 Spark Load 将数据加载到使用表达式分区的表中。
- 当使用`ALTER TABLE <table_name> DROP PARTITION <partition_name>`语句删除使用列表达式创建的分区时，该分区中的数据会被直接删除，且无法恢复。
- 目前，您无法对使用表达式分区创建的分区进行[备份和还原](../administration/Backup_and_restore.md)。