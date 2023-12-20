---
displayed_sidebar: English
---

# 动态分区

StarRocks 支持动态分区功能，能够自动管理分区的生命周期（TTL），例如自动分区表中新输入的数据以及删除过期的分区。这一特性显著降低了维护成本。

## 启用动态分区

以 table `site_access` 为例。要启用动态分区，您需要配置 PROPERTIES 参数。有关配置项的信息，请参阅[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

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

**`属性`**：

|参数|必填|描述|
|---|---|---|
|dynamic_partition.enable|No|启用动态分区。有效值为 TRUE 和 FALSE。默认值为 TRUE。|
|dynamic_partition.time_unit|是|动态创建分区的时间粒度。这是必需的参数。有效值为 HOUR、DAY、WEEK、MONTH 和 YEAR。时间粒度决定了动态创建分区的后缀格式。如果值为DAY，则动态创建分区的后缀格式为yyyyMMdd。示例分区名称后缀为 20200321。如果值为 WEEK，则动态创建分区的后缀格式为 yyyy_ww，例如 2020_13 表示 2020 年第 13 周。如果值为 MONTH，则动态创建分区的后缀格式为 yyyyMM，例如202003。如果值为YEAR，则动态创建分区的后缀格式为yyyy，例如2020。|
|dynamic_partition.time_zone|否|动态分区的时区，默认与系统时区相同。|
|dynamic_partition.start|No|动态分区的起始偏移量。该参数的值必须是负整数。在此偏移量之前的分区将根据参数dynamic_partition.time_unit的值确定的当前日、周或月进行删除。默认值为Integer.MIN_VALUE，即-2147483648，表示不会删除历史分区。|
|dynamic_partition.end|Yes|动态分区的结束偏移量。该参数的值必须是正整数。将提前创建从当前日、周或月到结束偏移量的分区。|
|dynamic_partition.prefix|否|添加到动态分区名称的前缀。默认值为 p。|
|dynamic_partition.buckets|No|每个动态分区的存储桶数量。默认值与保留字 BUCKETS 确定的桶数或 StarRocks 自动设置的桶数相同。|
|dynamic_partition.history_partition_num|否|动态分区机制创建的历史分区数量，默认值为0。当该值大于0时，会提前创建历史分区。从v2.5.2开始，StarRocks支持该参数。|
|dynamic_partition.start_day_of_week|否|当dynamic_partition.time_unit为WEEK时，该参数用于指定每周的第一天。有效值：1 到 7。1 表示星期一，7 表示星期日。默认值为 1，这意味着每周从星期一开始。|
|dynamic_partition.start_day_of_month|否|当dynamic_partition.time_unit为MONTH时，该参数用于指定每月的第一天。有效值：1 到 28。1 表示每月 1 日，28 表示每月 28 日。默认值为 1，表示每个月从 1 号开始。第一天不能是 29 日、30 日或 31 日。|
|dynamic_partition.replication_num|否|动态创建的分区中平板电脑的副本数量。默认值与创建表时配置的副本数相同。|

**FE 配置:**

dynamic_partition_check_interval_seconds：动态分区调度检查的时间间隔。默认值为 600 秒，即系统每 10 分钟检查一次分区情况，以确定分区是否满足 PROPERTIES 中指定的动态分区条件。如果不满足，系统将自动创建和删除分区。

## 查看分区

为表启用动态分区后，输入数据会持续自动进行分区。您可以使用以下 SQL 语句来查看当前的分区情况。例如，如果今天是 2020-03-25，您将只能看到从 2020-03-22 到 2020-03-28 时间范围内的分区。

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

您可以使用[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)语句来修改动态分区的属性，例如禁用动态分区。以下示例语句显示了如何进行修改。

```SQL
ALTER TABLE site_access 
SET("dynamic_partition.enable"="false");
```

> 注意：
- 要检查表的动态分区属性，请执行[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)语句。
- 您也可以使用 ALTER TABLE 语句来修改表的其他属性。
