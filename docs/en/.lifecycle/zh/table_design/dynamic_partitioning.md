---
displayed_sidebar: English
---

# 动态分区

StarRocks 支持动态分区，可以自动管理分区的生命周期（TTL），例如对表中的新输入数据进行分区和删除过期分区。此功能可显著降低维护成本。

## 启用动态分区

以表 `site_access` 为例。要启用动态分区，您需要配置 PROPERTIES 参数。有关配置项的信息，请参阅 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day)(
    PARTITION p20200321 VALUES LESS THAN ("2020-03-22"),
    PARTITION p20200322 VALUES LESS THAN ("2020-03-23"),
    PARTITION p20200323 VALUES LESS THAN ("2020-03-24"),
    PARTITION p20200324 VALUES LESS THAN ("2020-03-25")
)
DISTRIBUTED BY HASH(event_day, site_id)
PROPERTIES(
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-3",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.buckets" = "32",
    "dynamic_partition.history_partition_num" = "0"
);
```

**`PROPERTIES`**:

| 参数                               | 必填 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|-----------------------------------------| -------- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| dynamic_partition.enable                | 否       | 启用动态分区。有效值为 `TRUE` 和 `FALSE`。默认值为 `TRUE`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| dynamic_partition.time_unit             | 是      | 动态创建分区的时间粒度。这是一个必需参数。有效值为 `HOUR`、`DAY`、`WEEK`、`MONTH` 和 `YEAR`。时间粒度决定了动态创建分区的后缀格式。<ul><li>如果值为 `DAY`，则动态创建分区的后缀格式为 yyyyMMdd。例如，分区名称后缀为 `20200321`。</li><li>如果值为 `WEEK`，则动态创建分区的后缀格式为 yyyy_ww，例如 2020 年的第 13 周为 `2020_13`。</li><li>如果值为 `MONTH`，则动态创建分区的后缀格式为 yyyyMM，例如 `202003`。</li><li>如果值为 `YEAR`，则动态创建分区的后缀格式为 yyyy，例如 `2020`。</li></ul> |
| dynamic_partition.time_zone             | 否       | 动态分区的时区，默认与系统时区相同。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| dynamic_partition.start                 | 否       | 动态分区的起始偏移量。此参数的值必须为负整数。在当前日期、周或月的基础上，将删除该偏移量之前的分区，该日期、周或月由参数 `dynamic_partition.time_unit` 的值确定。默认值为 `Integer.MIN_VALUE`，即 -2147483648，表示不会删除历史分区。                                                                                                                                                                                                                                                                                                                                                                 |
| dynamic_partition.end                   | 是      | 动态分区的结束偏移量。此参数的值必须为正整数。将提前创建从当前日期、周或月到结束偏移量的分区。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| dynamic_partition.prefix                | 否       | 添加到动态分区名称的前缀。默认值为 `p`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.buckets               | 否       | 每个动态分区的存储桶数。默认值与保留字 BUCKETS 确定的数量或 StarRocks 自动设置的数量相同。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dynamic_partition.history_partition_num | 否       | 动态分区机制创建的历史分区数，默认值为 `0`。当该值大于 0 时，将提前创建历史分区。从 v2.5.2 版本开始，StarRocks 支持此参数。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| dynamic_partition.start_day_of_week     | 否       | 当 `dynamic_partition.time_unit` 为 `WEEK` 时，用于指定每周的第一天。有效值为 `1` 到 `7`。`1` 表示星期一，`7` 表示星期日。默认值为 `1`，表示每周从星期一开始。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| dynamic_partition.start_day_of_month    | 否       | 当 `dynamic_partition.time_unit` 为 `MONTH` 时，用于指定每月的第一天。有效值为 `1` 到 `28`。`1` 表示每月的第一天，`28` 表示每月的第 28 天。默认值为 `1`，表示每月从第一天开始。第一天不能是 29、30 或 31 号。                                                                                                                                                                                                                                                                                                                                                                                                                               |
| dynamic_partition.replication_num       | 否       | 动态创建的分区中每个 tablet 的副本数。默认值与创建表时配置的副本数相同。  |

**前端配置：**

`dynamic_partition_check_interval_seconds`：调度动态分区的时间间隔。默认值为 600 秒，表示每 10 分钟检查一次分区情况，以查看分区是否满足 `PROPERTIES` 中指定的动态分区条件。否则，将自动创建和删除分区。

## 查看分区

启用表的动态分区后，输入数据将持续自动进行分区。您可以使用以下语句查看当前分区。例如，如果当前日期为 2020-03-25，则只能看到 2020-03-22 到 2020-03-28 时间范围内的分区。

```SQL
SHOW PARTITIONS FROM site_access;

[types: [DATE]; keys: [2020-03-22]; ‥types: [DATE]; keys: [2020-03-23]; )
[types: [DATE]; keys: [2020-03-23]; ‥types: [DATE]; keys: [2020-03-24]; )
[types: [DATE]; keys: [2020-03-24]; ‥types: [DATE]; keys: [2020-03-25]; )
[types: [DATE]; keys: [2020-03-25]; ‥types: [DATE]; keys: [2020-03-26]; )
[types: [DATE]; keys: [2020-03-26]; ‥types: [DATE]; keys: [2020-03-27]; )
[types: [DATE]; keys: [2020-03-27]; ‥types: [DATE]; keys: [2020-03-28]; )
[types: [DATE]; keys: [2020-03-28]; ‥types: [DATE]; keys: [2020-03-29]; )
```

## 修改动态分区属性

您可以使用 [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 语句修改动态分区的属性，例如禁用动态分区。以下是一个示例语句。

```SQL
ALTER TABLE site_access 
SET("dynamic_partition.enable"="false");
```

> 注意：
>
> - 要查看表的动态分区属性，请执行 [SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) 语句。
> - 您还可以使用 ALTER TABLE 语句修改表的其他属性。
