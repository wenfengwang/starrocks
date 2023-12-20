---
displayed_sidebar: English
---

# 表达式分区（推荐）

自 v3.0 起，StarRocks 支持表达式分区（原称为自动分区），这种方式更加灵活且用户友好。该分区方法适用于多数场景，如基于连续时间范围或枚举值查询和管理数据。

您只需在创建表时指定一个简单的分区表达式（时间函数表达式或列表达式）。在数据加载过程中，StarRocks 将根据数据和分区表达式中定义的规则自动创建分区。您无需在创建表时手动创建大量分区，也无需配置动态分区属性。

## 基于时间函数表达式的分区

如果您经常基于连续时间范围查询和管理数据，只需在时间函数表达式中指定日期类型（DATE 或 DATETIME）列作为分区列，并指定年、月、日或小时作为分区粒度。StarRocks 将自动创建分区，并根据加载的数据和分区表达式设置分区的起始和结束日期或日期时间。

然而，在某些特殊场景中，例如将历史数据按月分区，而将近期数据按天分区，您必须使用[范围分区](./Data_distribution.md#range-partitioning)来创建分区。

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

|参数|必填|描述|
|---|---|---|
|`expression`|是|目前只支持 [date_trunc](../sql-reference/sql-functions/date-time-functions/date_trunc.md) 和 [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md) 函数。使用 `time_slice` 函数时，无需传递 `boundary` 参数，因为在此场景中，默认且有效的值为 `floor`，且该值不能为 `ceil`。|
|`time_unit`|是|分区粒度，可为 `hour`、`day`、`month` 或 `year`。不支持 `week` 分区粒度。若分区粒度为 `hour`，分区列必须为 DATETIME 数据类型，不能为 DATE 数据类型。|
|`partition_column`|是|分区列的名称。<br/><ul><li>分区列只能为 DATE 或 DATETIME 数据类型，且允许 `NULL` 值。</li><li>若使用 `date_trunc` 函数，分区列可以为 DATE 或 DATETIME 数据类型。若使用 `time_slice` 函数，分区列必须为 DATETIME 数据类型。</li><li>若分区列为 DATE 数据类型，支持的范围是 [0000-01-01 ~ 9999-12-31]。若分区列为 DATETIME 数据类型，支持的范围是 [0000-01-01 01:01:01 ~ 9999-12-31 23:59:59]。</li><li>目前，只能指定一个分区列，不支持多个分区列。</li></ul>|
|`partition_live_number`|否|要保留的最新分区数量。“最近”指的是分区按时间顺序排序，**以当前日期为基准**，保留倒数的分区数量，其余分区（更早创建的分区）被删除。StarRocks 通过调度任务来管理分区数量，调度间隔可通过 FE 动态参数 `dynamic_partition_check_interval_seconds` 配置，默认为 600 秒（10 分钟）。假设当前日期为 2023 年 4 月 4 日，`partition_live_number` 设置为 `2`，分区包括 `p20230401`、`p20230402`、`p20230403`、`p20230404`。则保留分区 `p20230403` 和 `p20230404`，其余分区被删除。若加载了脏数据，如未来日期 4 月 5 日和 4 月 6 日的数据，分区包括 `p20230401`、`p20230402`、`p20230403`、`p20230404`、`p20230405` 和 `p20230406`。则保留分区 `p20230403`、`p20230404`、`p20230405` 和 `p20230406`，其余分区被删除。|

### 使用说明

- 在数据加载过程中，StarRocks 会根据加载的数据自动创建一些分区，但如果加载作业因某种原因失败，StarRocks 自动创建的分区无法自动删除。
- StarRocks 将自动创建的分区数量上限默认设置为 4096，可通过 FE 参数 `max_automatic_partition_number` 配置。此参数可防止意外创建过多分区。
- 分区的命名规则与动态分区的命名规则一致。

### **示例**

示例 1：假设您经常按天查询数据。您可以使用分区表达式 `date_trunc()` 并在表创建时将分区列设置为 `event_day`，分区粒度设置为 `day`。在加载数据时，数据会根据日期自动分区。同一天的数据存储在同一个分区中，可以使用分区修剪显著提高查询效率。

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

例如，当加载以下两条数据时，StarRocks 将自动创建两个分区 `p20230226` 和 `p20230227`，范围分别为 [2023-02-26 00:00:00, 2023-02-27 00:00:00) 和 [2023-02-27 00:00:00, 2023-02-28 00:00:00)。如果后续加载的数据落在这些范围内，它们将自动路由到相应的分区。

```SQL
-- 插入两条数据
INSERT INTO site_access1  
    VALUES ("2023-02-26 20:12:04",002,"New York","Sam Smith",1),
           ("2023-02-27 21:06:54",001,"Los Angeles","Taylor Swift",1);

-- 查看分区
mysql > SHOW PARTITIONS FROM site_access1;
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range                                                                                                | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| 17138       | p20230226     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-26 00:00:00]; ..types: [DATETIME]; keys: [2023-02-27 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
| 17113       | p20230227     | 2              | 2023-07-19 17:53:59 | 0                  | NORMAL | event_day    | [types: [DATETIME]; keys: [2023-02-27 00:00:00]; ..types: [DATETIME]; keys: [2023-02-28 00:00:00]; ) | event_day, site_id | 6       | 3              | HDD           | 9999-12-31 23:59:59 | NULL                     | 0B       | false      | 0        |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+------------------------------------------------------------------------------------------------------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
2 行记录集 (0.00 秒)
```

示例 2：如果您想实现分区生命周期管理，即只保留一定数量的最新分区并删除历史分区，可以使用 `partition_live_number` 属性指定要保留的分区数量。

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

示例 3：假设您经常按周查询数据。您可以使用分区表达式 `time_slice()` 并在表创建时将分区列设置为 `event_day`，分区粒度设置为七天。一周的数据存储在一个分区中，可以使用分区修剪显著提高查询效率。

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

```
### 句法

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
|`partition_columns`|是|分区列的名称。<br/>分区列值可以是字符串（不支持 BINARY）、日期或日期时间、整数和布尔值。分区列允许 `NULL` 值。<br/>每个分区只能包含分区列具有相同值的数据。要在一个分区中包含分区列具有不同值的数据，请参阅[List partitioning](./list_partitioning.md)。|
|`partition_live_number`|否|要保留的分区数量。比较分区间分区列的值，定期删除值较小的分区，保留值较大的分区。<br/>StarRocks 通过调度任务来管理分区数量，调度间隔可以通过 FE 动态参数 `dynamic_partition_check_interval_seconds` 配置，默认为 600 秒（10 分钟）。<br/>**注意**<br/>如果分区列中的值为字符串，StarRocks 会比较分区名称的字典顺序，并定期保留较早的分区，同时删除较晚的分区。|

### 使用说明

- 在数据加载过程中，StarRocks 会根据加载的数据自动创建一些分区，但如果加载作业由于某种原因失败，StarRocks 自动创建的分区将无法自动删除。
- StarRocks 设置默认最大自动创建分区数为 4096，可以通过 FE 参数 `max_automatic_partition_number` 进行配置。该参数可以防止您意外创建太多分区。
- 分区的命名规则：如果指定多个分区列，则分区名称中不同分区列的值用下划线 `_` 连接，格式为 `p<分区列1中的值>_<分区列2中的值>_...`。例如，如果指定 `dt` 和 `province` 两列为分区列，均为字符串类型，加载值为 `2022-04-01` 和 `beijing` 的数据行，则自动创建的对应分区名为 `p20220401_beijing`。

### 示例

示例 1：假设您经常根据时间范围和特定城市查询数据中心账单明细。在创建表时，您可以使用分区表达式将第一个分区列指定为 `dt` 和 `city`。这样，属于相同日期和城市的数据就会被路由到同一个分区中，并且可以通过分区剪枝来显著提高查询效率。

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

将单个数据行插入表中。

```SQL
INSERT INTO t_recharge_detail1 
    VALUES (1, 1, 1, 'Houston', '2022-04-01');
```

查看分区。结果显示 StarRocks 根据加载的数据自动创建了分区 `p20220401_Houston`。在后续加载过程中，分区列 `dt` 和 `city` 中值为 `2022-04-01` 和 `Houston` 的数据将存储在该分区中。

> **注意**
> 每个分区只能包含为分区列指定一个值的数据。要为分区中的分区列指定多个值，请参阅[List partitioning](./list_partitioning.md)。

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

示例 2：您还可以在创建表时配置 "`partition_live_number`" 属性进行分区生命周期管理，例如指定表只能保留 3 个分区。

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
    "partition_live_number" = "3" -- 仅保留最近的三个分区
);
```

## 管理分区

### 将数据加载到分区中

在数据加载过程中，StarRocks 会根据加载的数据和由分区表达式定义的分区规则自动创建分区。

请注意，如果您在创建表时使用表达式分区，并且需要使用 [INSERT OVERWRITE](../loading/InsertInto.md#overwrite-data-via-insert-overwrite-select) 覆盖特定分区中的数据，无论该分区是否已创建，当前都需要在 `PARTITION()` 中显式提供一个分区范围。这与 [Range Partitioning](./Data_distribution.md#range-partitioning) 或 [List Partitioning](./list_partitioning.md) 不同，后者仅允许您在 `PARTITION(<partition_name>)` 中提供分区名称。

如果您在建表时使用时间函数表达式并希望覆盖特定分区中的数据，则需要提供该分区的开始日期或日期时间（建表时配置的分区粒度）。如果分区不存在，可以在数据加载时自动创建。

```SQL
INSERT OVERWRITE site_access1 PARTITION(event_day='2022-06-08 20:12:04')
    SELECT * FROM site_access2 PARTITION(p20220608);
```

如果您在建表时使用列表达式并希望覆盖特定分区中的数据，则需要提供该分区包含的分区列值。如果分区不存在，可以在数据加载时自动创建。

```SQL
INSERT OVERWRITE t_recharge_detail1 PARTITION(dt='2022-04-02',city='texas')
    SELECT * FROM t_recharge_detail2 PARTITION(p20220402_texas);
```

### 查看分区

当您想要查看自动创建的分区的具体信息时，需要使用 `SHOW PARTITIONS FROM <table_name>` 语句。`SHOW CREATE TABLE <table_name>` 语句仅返回在创建表时配置的表达式分区语法。

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

- 从 v3.1.0 开始，StarRocks 的 [共享数据模式](../deployment/shared_data/s3.md) 支持 [时间函数表达式](#partitioning-based-on-a-time-function-expression)。而从 v3.1.1 开始，StarRocks 的 [共享数据模式](../deployment/shared_data/s3.md) 进一步支持 [列表达式](#partitioning-based-on-the-column-expression-since-v31)。
- 目前，不支持使用 CTAS 创建配置表达式分区的表。
- 目前，不支持使用 Spark Load 将数据加载到使用表达式分区的表中。
- 使用 `ALTER TABLE <table_name> DROP PARTITION <partition_name>` 语句删除通过列表达式创建的分区时，分区中的数据将被直接删除，且无法恢复。