---
displayed_sidebar: English
---

# 表达式分区（推荐）

自 v3.0 起，StarRocks 支持表达式分区（曾称为自动分区），这种方式更加灵活和用户友好。此分区方法适合大多数场景，比如基于连续时间范围或枚举值来查询和管理数据。

您只需在创建表时指定一个简单的分区表达式（可以是时间函数表达式或列表达式）。在数据加载过程中，StarRocks 会根据数据和分区表达式中定义的规则自动创建分区。这样，您就无需在创建表时手动创建众多分区，也不需要配置动态分区属性。

## 基于时间函数表达式的分区

如果您经常需要根据连续时间范围来查询和管理数据，只需在时间函数表达式中指定一个日期类型（DATE 或 DATETIME）列作为分区列，并指定年、月、日或小时作为分区粒度。StarRocks 将自动创建分区，并根据加载的数据和分区表达式设置分区的起始和结束日期或时间。

然而，在一些特殊场景下，如需将历史数据按月分区、近期数据按日分区，您必须使用[范围分区](./Data_distribution.md#range-partitioning)来创建分区。

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

|参数|必填|说明|
|---|---|---|
|表达式|YES|当前仅支持 date_trunc 和 time_slice 函数。如果使用函数time_slice，则不需要传递边界参数。因为在这种场景下，该参数的默认有效值为floor，不能为ceil。|
|time_unit|YES|分区粒度，可以是小时、天、月或年。不支持周分区粒度。如果分区粒度为小时，则分区列必须为DATETIME数据类型，且不能为DATE数据类型。|
|partition_column|YES|分区列的名称。分区列只能是 DATE 或 DATETIME 数据类型。分区列允许 NULL 值。如果使用 date_trunc 函数，分区列可以是 DATE 或 DATETIME 数据类型。如果使用 time_slice 函数，分区列必须是 DATETIME 数据类型。如果分区列为DATE数据类型，支持的范围为[0000-01-01 ~ 9999-12-31]。如果分区列为DATETIME数据类型，支持的范围为[0000-01-01 01:01:01 ~ 9999-12-31 23:59:59]。目前只能指定一个分区列，可以指定多个分区列。不支持分区列。|
|partition_live_number|NO|要保留的最新分区的数量。 “最近”是指分区按照时间顺序排序，以当前日期为基准，保留倒数的分区数量，其余分区（更早创建的分区）被删除。 StarRocks通过调度任务来管理分区数量，调度间隔可以通过FE动态参数dynamic_partition_check_interval_seconds进行配置，默认为600秒（10分钟）。假设当前日期为2023年4月4日，partition_live_number设置为2，分区包括p20230401、p20230402、p20230403、p20230404。保留分区p20230403和p20230404，删除其他分区。如果加载脏数据，例如未来日期4月5日、4月6日的数据，则分区为p20230401、p20230402、p20230403、p20230404、p20230405、p20230406。然后保留分区 p20230403、p20230404、p20230405 和 p20230406，并删除其他分区。|

### 使用说明

- 在数据加载过程中，StarRocks 会根据加载的数据自动创建一些分区，但如果加载作业因某些原因失败，StarRocks 自动创建的分区无法自动删除。
- StarRocks 默认最大自动创建分区数量设为 4096，可通过 FE 参数 max_automatic_partition_number 进行配置。此参数可防止不慎创建过多分区。
- 分区的命名规则与动态分区的命名规则相同。

### **示例**

示例 1：假设您经常需要按天查询数据。您可以使用分区表达式 date_trunc()，在创建表时将分区列设置为 event_day，分区粒度设置为天。数据在加载时会根据日期自动分区，同一天的数据存储在同一分区中，这样可以通过分区剪枝显著提高查询效率。

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

例如，当加载以下两行数据时，StarRocks 将自动创建两个分区，p20230226 和 p20230227，范围分别是 [2023-02-26 00:00:00, 2023-02-27 00:00:00) 和 [2023-02-27 00:00:00, 2023-02-28 00:00:00)。如果后续加载的数据落在这些范围内，它们将自动路由到相应的分区。

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

示例 2：如果您想实施分区生命周期管理，即只保留一定数量的最新分区并删除历史分区，您可以使用 partition_live_number 属性来指定保留的分区数量。

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

示例 3：假设您经常需要按周查询数据。您可以使用分区表达式 time_slice()，在创建表时将分区列设置为 event_day，分区粒度设置为七天。一周的数据存储在一个分区中，通过分区剪枝可以显著提高查询效率。

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

## 基于列表达式的分区（自 v3.1 起）

如果您经常查询和管理特定类型的数据，只需将代表该类型的列指定为分区列。StarRocks 将根据加载数据的分区列值自动创建分区。

但在一些特殊场景下，如当表中包含`城市`列，并且您经常根据国家和城市查询和管理数据时，您必须使用[list partitioning](./list_partitioning.md)将同一国家的多个城市的数据存储在同一分区中。

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

|参数|必填|说明|
|---|---|---|
|partition_columns|YES|分区列的名称。分区列值可以是字符串（不支持 BINARY）、日期或日期时间、整数和布尔值。分区列允许 NULL 值。每个分区只能包含分区列具有相同值的数据。要在分区的分区列中包含具有不同值的数据，请参阅列表分区。|
|partition_live_number|No|要保留的分区数量。比较分区间分区列的值，定期删除值较小的分区，保留值较大的分区。StarRocks通过调度任务来管理分区数量，调度间隔可以通过FE动态参数dynamic_partition_check_interval_seconds配置，默认为600 秒（10 分钟）。注意如果分区列中的值为字符串，StarRocks 会比较分区名称的字典顺序，并定期保留较早的分区，同时删除较晚的分区。|

### 使用说明

- 在数据加载过程中，StarRocks 会根据加载的数据自动创建一些分区，但如果加载作业因某些原因失败，StarRocks 自动创建的分区无法自动删除。
- StarRocks 默认最大自动创建分区数量设为 4096，可通过 FE 参数 max_automatic_partition_number 进行配置。此参数可防止不慎创建过多分区。
- 如果指定了多个分区列，分区名称将以不同分区列值之间的下划线连接，格式为 p<分区列1的值>_<分区列2的值>_...。例如，如果指定了 dt 和 province 两个分区列，均为字符串类型，加载了值为 2022-04-01 和 beijing 的数据行，则自动创建的对应分区名为 p20220401_beijing。

### 示例

示例 1：假设您经常根据时间范围和特定城市查询数据中心的账单明细。在创建表时，您可以使用分区表达式来指定 dt 和 city 作为第一和第二分区列。这样，属于同一日期和城市的数据会被路由到同一分区中，通过分区剪枝可以显著提高查询效率。

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

向表中插入单条数据行。

```SQL
INSERT INTO t_recharge_detail1 
    VALUES (1, 1, 1, 'Houston', '2022-04-01');
```

查看分区。结果显示 StarRocks 根据加载的数据自动创建了分区 p20220401_Houston1。在后续加载过程中，分区列 dt 和 city 的值为 2022-04-01 和 Houston 的数据将被存储在该分区中。

> **注意**
> 每个分区只能包含具有指定单一分区列值的数据。要在一个分区中指定分区列的多个值，请参考[列表分区](./list_partitioning.md)。

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

示例 2：您还可以在创建表时配置 partition_live_number 属性以进行分区生命周期管理，例如指定表只应保留 3 个分区。

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

### 将数据加载到分区

在数据加载过程中，StarRocks 将根据加载的数据和分区表达式定义的规则自动创建分区。

请注意，如果您在创建表时使用了表达式分区，并且需要使用 [INSERT OVERWRITE](../loading/InsertInto.md#overwrite-data-via-insert-overwrite-select) 覆盖特定分区中的数据，无论该分区是否已存在，当前都需要在 `PARTITION()` 中明确提供一个分区范围。这与[范围分区](./Data_distribution.md#range-partitioning)或[列表分区](./list_partitioning.md)不同，后者只需要在 `PARTITION (
<partition_name>)` 中提供分区名称。

如果您在创建表时使用了时间函数表达式，并希望覆盖特定分区中的数据，您需要提供该分区的起始日期或时间（即创建表时配置的分区粒度）。如果分区不存在，它可以在数据加载过程中自动创建。

```SQL
INSERT OVERWRITE site_access1 PARTITION(event_day='2022-06-08 20:12:04')
    SELECT * FROM site_access2 PARTITION(p20220608);
```

如果您在创建表时使用了列表达式，并希望覆盖特定分区中的数据，您需要提供该分区包含的分区列值。如果分区不存在，它也可以在数据加载过程中自动创建。

```SQL
INSERT OVERWRITE t_recharge_detail1 PARTITION(dt='2022-04-02',city='texas')
    SELECT * FROM t_recharge_detail2 PARTITION(p20220402_texas);
```

### 查看分区

当您想要查看自动创建的分区的具体信息时，您需要使用 SHOW PARTITIONS FROM <table_name> 语句。SHOW CREATE TABLE <table_name> 语句仅返回在创建表时配置的表达式分区的语法。

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

- 自 v3.1.0 起，StarRocks的[共享数据模式](../deployment/shared_data/s3.md)支持[时间函数表达式](#partitioning-based-on-a-time-function-expression)。而自v3.1.1起，StarRocks的[共享数据模式](../deployment/shared_data/s3.md)进一步支持[列表达式](#partitioning-based-on-the-column-expression-since-v31)。
- 目前，不支持使用 CTAS（Create Table As Select）创建配置了表达式分区的表。
- 目前，不支持使用 Spark Load 向使用表达式分区的表中加载数据。
- 使用 ALTER TABLE <table_name> DROP PARTITION <partition_name> 语句删除通过列表达式创建的分区时，分区中的数据将被直接移除，且无法恢复。
- 目前您不能[备份和恢复](../administration/Backup_and_restore.md)通过表达式分区创建的分区。
