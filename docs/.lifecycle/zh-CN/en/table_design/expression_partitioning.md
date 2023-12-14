---
displayed_sidebar: "English"
---

# 表达式分区（推荐）

自 v3.0 版本开始，StarRocks 支持表达式分区（先前称为自动分区），这种分区方式更加灵活和用户友好。此分区方法适用于大多数场景，如基于连续时间范围或枚举值查询和管理数据。

您只需要在表创建时指定一个简单的分区表达式（可以是时间函数表达式或列表达式）。在数据加载期间，StarRocks 将根据数据和分区表达式定义的规则自动创建分区。您不再需要在表创建时手动创建大量分区，也不需要配置动态分区属性。

## 基于时间函数表达式的分区

如果您经常根据连续时间范围查询和管理数据，只需将日期类型（DATE 或 DATETIME）列指定为分区列，然后在时间函数表达式中指定年、月、日或小时作为分区粒度即可。StarRocks 将根据加载的数据和分区表达式自动创建分区，并设置分区的开始和结束日期或日期时间。

但是，在某些特殊场景下，例如将历史数据按月分区，最近数据按日分区，您必须使用[范围分区](./Data_distribution.md#range-partitioning)来创建分区。

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

| 参数                      | 是否必需  | 描述                                                       |
| ----------------------- | -------- | ------------------------------------------------------------ |
| `expression`            |     是     | 当前仅支持[date_trunc](../sql-reference/sql-functions/date-time-functions/date_trunc.md)和[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)函数。如果使用`time_slice`函数，则无需传递`boundary`参数。因为在这种情况下，此参数的默认有效值是`floor`，不允许值为`ceil`。 |
| `time_unit`             |       是   | 分区粒度，可以是 `hour`、`day`、`month` 或 `year`。不支持分区粒度为 `week`。如果分区粒度为 `hour`，分区列必须是 DATETIME 数据类型，不能是 DATE 数据类型。 |
| `partition_column` |     是     | 分区列的名称。<br/><ul><li>分区列只能是 DATE 或 DATETIME 数据类型。分区列允许 `NULL` 值。</li><li>如果使用 `date_trunc` 函数，分区列可以是 DATE 或 DATETIME 数据类型。如果使用 `time_slice` 函数，则分区列必须是 DATETIME 数据类型。</li><li>如果分区列是 DATE 数据类型，支持的范围是[0000-01-01 ~ 9999-12-31]。如果分区列是 DATETIME 数据类型，支持的范围是[0000-01-01 01:01:01 ~ 9999-12-31 23:59:59]。</li><li>当前，只能指定一个分区列，不支持多个分区列。</li></ul> |
| `partition_live_number` |      否    | 要保留的最新分区的数量。“最新”指的是按时间顺序排序的分区，以当前日期为基准，向前计数的分区会被保留，而其他（早期创建的）分区会被删除。StarRocks安排计划来管理分区的数量，调度间隔可以通过FE 动态参数 `dynamic_partition_check_interval_seconds` 进行配置，默认为 600 秒（10分钟）。假设当前日期为2023年4月4日，`partition_live_number` 设置为 `2`，分区包括 `p20230401`、`p20230402`、`p20230403`、`p20230404`。分区 `p20230403` 和 `p20230404` 被保留，其他分区被删除。如果加载了脏数据，如来自未来日期4月5日和4月6日的数据，分区包括 `p20230401`、`p20230402`、`p20230403`、`p20230404`、`p20230405`和`p20230406`。那么分区 `p20230403`、`p20230404`、`p20230405` 和 `p20230406` 被保留，其他分区被删除。 |

### 使用注意事项

- 在数据加载期间，StarRocks根据加载的数据自动创建一些分区，但如果由于某种原因加载作业失败，则StarRocks自动创建的分区无法自动删除。
- StarRocks将默认最大自动创建分区数设置为4096，可以通过FE参数 `max_automatic_partition_number` 进行配置。此参数可以防止您意外创建过多分区。
- 分区的命名规则与动态分区的命名规则一致。

### **示例**

示例 1: 假设您经常按天查询数据。您可以使用分区表达式 `date_trunc()`，并在表创建时将分区列设置为 `event_day`，分区粒度设置为 `day`。数据在加载期间会基于日期自动进行分区，同一天的数据存储在一个分区中，可以使用分区修剪来显著提高查询效率。

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

例如，当加载以下两行数据时，StarRocks将自动创建两个分区 `p20230226` 和 `p20230227`，范围分别为 [2023-02-26 00:00:00, 2023-02-27 00:00:00) 和 [2023-02-27 00:00:00, 2023-02-28 00:00:00)。如果随后加载的数据在这些范围内，它们将自动路由到相应的分区。

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

示例 2: 如果您想要实施分区生命周期管理，即保留一定数量的最新分区并删除历史分区，可以使用`partition_live_number`属性指定要保留的分区数量。

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
    "partition_live_number" = "3" -- only retains the most recent three partitions
);
```

例3: 假设您经常根据日期查询数据。您可以使用分区表达式 `time_slice()` 并将分区列设置为 `event_day`，分区粒度为7天，创建表时。

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

## 基于列表达式的分区（自V3.1起）

如果您经常查询和管理特定类型的数据，只需指定表示类型的列作为分区列即可。StarRocks 将根据加载的数据的分区列值自动创建分区。

但是，在某些特殊场景下，例如表包含列 `city`，您经常根据国家和城市查询和管理数据。您必须使用 [列表分区](./list_partitioning.md) 将同一个国家的多个城市的数据存储在同一个分区。

### 语法

```sql
PARTITION BY expression
...
[ PROPERTIES("partition_live_number" = "xxx") ]

expression ::=
    ( partition_columns )
    
partition_columns ::=
    <colum>, [ <column> [,...] ]
```

### 参数

| **参数**         | **必需** | **描述**                                                     |
| -------------- | ------ | ----------------------------------------------------------- |
 | `partition_columns` | 是     | 分区列的名称。<br/> <ul><li>分区列的值可以是字符串（不支持 BINARY）、日期、日期时间、整数和布尔值。分区列允许 `NULL` 值。</li><li>每个分区只能包含具有相同分区列值的数据。要在同一个分区内包含分区列具有不同值的数据，请参见 [列表分区](./list_partitioning.md)。</li></ul> |
| `partition_live_number` | 否     | 要保留的分区数量。对比分区间分区列的值，定期删除值较小的分区以保留值较大的分区。<br/>StarRocks 定期调度任务管理分区数量，并且调度间隔可以通过 FE 动态参数 `dynamic_partition_check_interval_seconds` 配置， 默认为600秒（10分钟）。<br/>**注意**<br/>如果分区列中的值是字符串，StarRocks 比较分区名称的字典序，定期保留较早的分区并删除较晚的分区。 |

### 使用注意事项

- 在数据加载时，StarRocks 根据加载的数据自动创建一些分区，如果由于某种原因加载作业失败，则由 StarRocks 自动创建的分区不能自动删除。
- StarRocks 将默认最大自动创建分区数设置为 4096，可以通过 FE 参数 `max_automatic_partition_number` 进行配置。该参数可防止意外创建太多的分区。
- 分区的命名规则：如果指定了多个分区列，不同分区列的值通过下划线 `_` 连接。例如 `p<value in partition_column 1>_<value in partition_column 2>_...`。例如，如果指定了 `dt` 和 `province` 作为分区列，两者都是字符串类型，并且加载了值为 `2022-04-01` 和 `beijing` 的数据行，则自动创建的分区称为 `p20220401_beijing`。

### 示例

例1: 假设您经常根据时间范围和特定城市查询数据中心计费详情。创建表时，可以使用分区表达式指定第一个分区列为 `dt` 和 `city`。这样，属于同一日期和城市的数据将路由到同一个分区，可以使用分区修剪显著提高查询效率。

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

将单个数据行插入到表中。

```SQL
INSERT INTO t_recharge_detail1 
    VALUES (1, 1, 1, 'Houston', '2022-04-01');
```

查看分区。结果显示 StarRocks 基于加载的数据自动创建了一个名为 `p20220401_Houston1` 的分区。在随后的加载中，值为 `2022-04-01`和`Houston` 的数据在分区列 `dt` 和 `city` 中存储在该分区内。

> **注意**
>
> 每个分区只能包含指定分区列的一个值的数据。要在同一个分区列的分区中指定多个值，参见 [列表分区](./list_partitioning.md)。

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

例2: 您还可以在表创建时配置 `partition_live_number` 属性来进行分区生命周期管理，例如指定表仅保留3个分区。

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

在数据加载时，StarRocks 将根据加载的数据和分区规则自动创建分区。

请注意，如果您在表创建时使用表达式分区，并且需要使用 [INSERT OVERWRITE](../loading/InsertInto.md#overwrite-data-via-insert-overwrite-select) 覆盖特定分区中的数据，则当前需要在 `PARTITION()` 中明确提供分区范围。这与 [范围分区](./Data_distribution.md#range-partitioning) 或 [列表分区](./list_partitioning.md) 不同，范围分区 或列表分区允许仅在 `PARTITION (<partition_name>)` 中提供分区名称。

如果您在表创建时使用时间函数表达式，并且要覆盖特定分区中的数据，则需要提供该分区的开始日期或日期时间（表创建时配置的分区粒度）。如果分区不存在，可以在数据加载中自动创建。

```SQL
INSERT OVERWRITE site_access1 PARTITION(event_day='2022-06-08 20:12:04')
    SELECT * FROM site_access2 PARTITION(p20220608);
```

如果您在表创建时使用列表达式，并且要覆盖特定分区中的数据，则需要提供分区包含的分区列值。如果分区不存在，可以在数据加载中自动创建。

```SQL
INSERT OVERWRITE t_recharge_detail1 PARTITION(dt='2022-04-02',city='texas')
    SELECT * FROM t_recharge_detail2 PARTITION(p20220402_texas);
```

### 查看分区

要查看有关自动创建的分区的特定信息，需要使用 `SHOW PARTITIONS FROM <table_name>` 语句。`SHOW CREATE TABLE <table_name>` 语句仅返回在表创建时配置的表达式分区的语法。
| PartitionId | PartitionName      | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | List                        | DistributionKey | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 16890       | p20220401_休斯顿   | 2              | 2023-07-19 17:24:53 | 0                  | 正常   | dt, city     | (('2022-04-01', '休斯顿'))   | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
| 17056       | p20220402_德克萨斯 | 2              | 2023-07-19 17:27:42 | 0                  | 正常   | dt, city     | (('2022-04-02', '德克萨斯')) | id              | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 2.5KB    | false      | 1        |
+-------------+-------------------+----------------+---------------------+--------------------+--------+--------------+-----------------------------+-----------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
{R}
## 限制

- 自v3.1.0起，StarRocks的[共享数据模式](../deployment/shared_data/s3.md)支持[时间函数表达式](#基于时间函数表达式的分区)，并自v3.1.1起，StarRocks的[共享数据模式](../deployment/shared_data/s3.md)进一步支持[列表达式](#自v31起基于列表达式的分区)。
- 目前，不支持使用CTAS创建配置了表达式分区的表。
- 目前，不支持使用Spark Load将数据加载到使用表达式分区的表中。
- 当使用`ALTER TABLE <table_name> DROP PARTITION <partition_name>`语句删除使用列表达式创建的分区时，分区中的数据将被直接移除，无法恢复。
- 目前无法[备份和恢复](../administration/Backup_and_restore.md)使用表达式分区创建的分区。