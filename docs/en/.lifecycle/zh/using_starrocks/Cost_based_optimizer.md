---
displayed_sidebar: English
---

# 收集 CBO 统计信息

本主题介绍了StarRocks成本优化器（CBO）的基本概念，以及如何收集统计信息以供CBO选择最佳查询计划。StarRocks 2.4引入了直方图，用于收集准确的数据分布统计。

从v3.2.0开始，StarRocks支持从Hive、Iceberg和Hudi表中收集统计信息，减少对其他元存储系统的依赖。其语法类似于收集StarRocks内部表。

## 什么是CBO

CBO对于查询优化至关重要。当SQL查询到达StarRocks时，它会被解析为逻辑执行计划。CBO会将逻辑计划重写并转换为多个物理执行计划。然后，CBO会估算计划中每个操作符的执行成本（例如CPU、内存、网络和I/O），并选择成本最低的查询路径作为最终的物理计划。

StarRocks CBO在StarRocks 1.16.0版本中推出，并从1.19版本开始默认启用。StarRocks CBO是基于Cascades框架开发的，可以根据各种统计信息估算成本。它能够在数以万计的执行计划中选择成本最低的执行计划，从而显著提高复杂查询的效率和性能。

统计信息对于CBO非常重要。它们决定了成本估算是否准确且有用。以下各节详细介绍了统计信息的类型、收集策略以及如何收集统计信息和查看统计信息。

## 统计信息的类型

StarRocks会收集各种统计数据作为成本估算的输入。

### 基本统计

默认情况下，StarRocks会定期收集表和列的以下基本统计信息：

- row_count：表中的总行数

- data_size：列的数据大小

- ndv：列的基数，即列中的不同值数量

- null_count：列中的NULL值数量

- min：列中的最小值

- max：列中的最大值

完整的统计信息存储在`_statistics_.column_statistics`数据库的`column_statistics`表中。您可以查看此表以查询表统计信息。以下是从该表中查询统计数据的示例。

```sql
SELECT * FROM _statistics_.column_statistics\G
*************************** 1. row ***************************
      table_id: 10174
  partition_id: 10170
   column_name: is_minor
         db_id: 10169
    table_name: zj_test.source_wiki_edit
partition_name: p06
     row_count: 2
     data_size: 2
           ndv: NULL
    null_count: 0
           max: 1
           min: 0
```

### 直方图

StarRocks 2.4引入了直方图来补充基础统计。直方图被认为是一种有效的数据表示方式。对于数据偏斜的表，直方图可以准确反映数据分布。

StarRocks使用等高直方图，这些直方图由多个桶构建而成。每个桶包含等量的数据。对于频繁查询且对选择性有重大影响的数据值，StarRocks会为其分配单独的桶。存储桶越多，估算就越准确，但也可能导致内存使用量略微增加。您可以调整直方图采集任务的桶数量和最常见值（MCV）。

**直方图适用于数据高度偏斜且查询频繁的列。如果表数据是均匀分布的，则无需创建直方图。只能在数值、DATE、DATETIME或字符串类型的列上创建直方图。**

目前，StarRocks仅支持手动收集直方图。直方图存储在`_statistics_.histogram_statistics`数据库的`histogram_statistics`表中。以下是从该表中查询统计数据的示例：

```sql
SELECT * FROM _statistics_.histogram_statistics\G
*************************** 1. row ***************************
   table_id: 10174
column_name: added
      db_id: 10169
 table_name: zj_test.source_wiki_edit
    buckets: NULL
        mcv: [["23","1"],["5","1"]]
update_time: 2023-12-01 15:17:10.274000
```

## 收集类型和方法

数据大小和数据分布在表中不断变化。必须定期更新统计信息以反映这些数据变化。在创建统计信息收集任务之前，您需要选择最适合您业务需求的收集类型和方法。

StarRocks支持全量收集和采样收集，两者都可以自动和手动执行。默认情况下，StarRocks会自动收集表的全量统计信息。它每5分钟检查一次数据更新。如果检测到数据更改，将自动触发数据收集。如果不想使用自动全量收集，可以将FE配置项`enable_collect_full_statistic`设置为`false`，并自定义收集任务。

| **收集类型** | **收集方法** | **描述**                                              | **优缺点**                               |
| ------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 全量收集     | 自动/手动      | 扫描整个表以收集统计信息。统计信息按分区收集。如果某个分区没有数据变化，则不会从该分区收集数据，从而减少资源消耗。完整的统计信息存储在`_statistics_.column_statistics`表中。 | 优点：统计数据准确，有助于CBO进行准确的估算。缺点：消耗系统资源，速度慢。从2.5版本开始，StarRocks支持设置自动收集周期，从而减少资源消耗。 |
| 采样收集  | 自动/手动      | 从每个表分区均匀提取`N`行数据。统计信息按表收集。每列的基本统计信息都存储为一条记录。列的基数信息（ndv）是根据采样数据估算的，不准确。采样统计信息存储在`_statistics_.table_statistic_v1`表中。 | 优点：系统资源消耗少，速度快。缺点：统计不完整，可能影响成本估算的准确性。 |

## 收集统计信息

StarRocks提供了灵活的统计信息收集方法。您可以选择自动、手动或自定义收集，以适应您的业务场景。

### 自动收集

对于基本统计信息，StarRocks默认情况下会自动收集表的全量统计信息，无需手动操作。对于未收集统计信息的表，StarRocks会在调度周期内自动收集统计信息。对于已收集统计信息的表，StarRocks会更新表中的总行数和修改后的行数，并定期保留这些信息，以判断是否触发自动收集。

从2.4.5版本开始，StarRocks允许您指定自动全量收集的收集周期，避免自动全量收集导致集群性能波动。此周期由FE参数`statistic_auto_analyze_start_time`和`statistic_auto_analyze_end_time`指定。

将触发自动收集的条件：

- 自上次统计信息收集以来，表数据已更改。

- 收集时间在配置的收集周期范围内。（默认收集周期为全天。

- 上一个收集作业的更新时间早于分区的最新更新时间。

- 表统计信息的健康状况低于指定的阈值（`statistic_auto_collect_ratio`）。

> 统计健康计算公式：
>
> 如果更新了数据的分区数小于10，则公式为`1 - (Number of updated rows since previous collection/Total number of rows)`。
> 如果更新了数据的分区数大于或等于10，则公式为`1 - MIN(Number of updated rows since previous collection/Total number of rows, Number of updated partitions since previous collection/Total number of partitions)`。

此外，StarRocks还允许您根据表大小和表更新频率配置收集策略：

- 对于数据量较小的表，即使表数据频繁更新，统计信息也会实时收集，没有限制。`statistic_auto_collect_small_table_size`参数可用于确定表是小表还是大表。您还可以使用`statistic_auto_collect_small_table_interval`来配置小表的收集间隔。

- 对于数据量较大的表，存在以下限制：

  - 默认收集间隔不小于12小时，可以使用`statistic_auto_collect_large_table_interval`进行配置。

  - 当达到收集间隔且统计信息健康状况低于自动采样收集的阈值（`statistic_auto_collect_sample_threshold`）时，将触发采样收集。

  - 当达到采集间隔且统计信息健康状况高于自动采样采集阈值（`statistic_auto_collect_sample_threshold`）且低于自动收集阈值（`statistic_auto_collect_ratio`）时，将触发全量收集。

  - 当采集数据的最大分区大小（`statistic_max_full_collect_data_size`）大于100GB时，将触发采样收集。

  - 仅对更新时间晚于上一次采集任务时间的分区进行统计。不收集没有数据更改的分区的统计信息。

:::提示

更改表数据后，手动触发该表的采样收集任务会使得该表的采样收集任务的更新时间晚于数据更新时间，不会触发该调度周期内该表的自动全量采集。
:::

默认情况下，自动全量收集处于启用状态，并由系统使用默认设置运行。

下表描述了默认设置。如果需要修改它们，请运行**ADMIN SET CONFIG**命令。

| **FE配置项**         | **类型** | **默认值** | **描述**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| enable_statistic_collect              | 布尔  | TRUE              | 是否收集统计信息。默认情况下，此开关处于打开状态。 |
| enable_collect_full_statistic         | 布尔  | TRUE              | 是否启用自动全量收集。默认情况下，此开关处于打开状态。 |
| statistic_collect_interval_sec        | 长     | 300               | 自动收集过程中检查数据更新的时间间隔。单位：秒。 |
| statistic_auto_analyze_start_time | 字符串      | 00:00:00   | 自动收集的开始时间。取值范围：`00:00:00` - `23:59:59`。 |
| statistic_auto_analyze_end_time | 字符串      | 23:59:59  | 自动收集的结束时间。取值范围：`00:00:00` - `23:59:59`。 |
| statistic_auto_collect_small_table_size     | 长    | 5368709120   | 用于确定表是否为自动全量收集小表的阈值。大小大于此值的表被视为大表，而大小小于或等于此值的表被视为小表。单位：字节。默认值：5368709120（5GB）。                         |
| statistic_auto_collect_small_table_interval | 长    | 0         | 自动收集小表全量统计信息的时间间隔。单位：秒。                              |

| statistic_auto_collect_large_table_interval | 长整型    | 43200        | 自动收集大表完整统计信息的时间间隔。单位：秒。默认值：43200（12小时）。                               |
| statistic_auto_collect_ratio          | 浮点型    | 0.8               | 用于确定自动收集统计信息是否正常的阈值。如果统计健康度低于此阈值，则会触发自动收集。 |
| statistic_auto_collect_sample_threshold  | 双精度 | 0.3   | 触发自动采样收集的统计健康度阈值。如果统计健康值低于此阈值，则触发自动采样收集。 |
| statistic_max_full_collect_data_size | 长整型      | 107374182400      | 自动收集的最大分区大小。单位：字节。默认值：107374182400（100GB）。如果分区超过此值，则放弃完整收集，改为执行采样收集。 |
| statistic_full_collect_buffer | 长整型 | 20971520 | 自动收集任务占用的最大缓冲区大小。单位：字节。默认值：20971520（20MB）。 |
| statistic_collect_max_row_count_per_query | 整型  | 5000000000        | 单个分析任务要查询的最大行数。如果超过此值，将拆分分析任务为多个查询。 |
| statistic_collect_too_many_version_sleep | 长整型 | 600000 | 自动收集任务的休眠时间（如果运行收集任务的表具有太多数据版本）。单位：毫秒。默认值：600000（10分钟）。  |

大多数统计信息收集可以依赖自动作业，但如果您有特定需求，可以通过执行ANALYZE TABLE语句手动创建任务，也可以通过执行CREATE ANALYZE语句自定义自动任务。

### 手动收集

您可以使用ANALYZE TABLE创建手动收集任务。默认情况下，手动收集是同步操作。您也可以将其设置为异步操作。在异步模式下，执行ANALYZE TABLE后，系统会立即返回该语句是否成功。但是，收集任务将在后台运行，您不必等待结果。您可以通过运行SHOW ANALYZE STATUS来检查任务的状态。异步收集适用于数据量较大的表，而同步收集适用于数据量较小的表。 **手动收集任务在创建后仅运行一次。您无需删除手动收集任务。**

#### 手动收集基本统计信息

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES (property [,property])]
```

参数说明：

- 收集类型
  - FULL：表示全量收集。
  - SAMPLE：表示采样收集。
  - 如果未指定收集类型，则默认使用完整收集。

- `col_name`：要收集统计信息的列。用逗号（,）分隔多个列。如果未指定此参数，则收集整个表。

- [WITH SYNC | ASYNC MODE]：指定手动收集任务以同步或异步模式运行。如果未指定此参数，则默认使用同步收集。

- `PROPERTIES`：自定义参数。如果未指定`PROPERTIES`，则使用`fe.conf`文件中的默认设置。可以通过`SHOW ANALYZE STATUS`输出中的`Properties`列查看实际使用的属性。

| **属性**                | **类型** | **默认值** | **描述**                                              |
| ----------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows | 整型      | 200000            | 采样收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。 |

示例

手动全量收集

```SQL
-- 使用默认设置手动收集表的完整统计信息。
ANALYZE TABLE tbl_name;

-- 使用默认设置手动收集表的完整统计信息。
ANALYZE FULL TABLE tbl_name;

-- 使用默认设置手动收集表中指定列的统计信息。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

手动采样收集

```SQL
-- 使用默认设置手动收集表的部分统计信息。
ANALYZE SAMPLE TABLE tbl_name;

-- 使用指定行数手动收集表中指定列的统计信息。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

#### 手动收集直方图

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
[PROPERTIES (property [,property])]
```

参数说明：

- `col_name`：要收集统计信息的列。用逗号（,）分隔多个列。如果未指定此参数，则收集整个表。对于直方图，此参数是必需的。

- [WITH SYNC | ASYNC MODE]：指定手动收集任务以同步或异步模式运行。如果未指定此参数，则默认使用同步收集。

- `WITH N BUCKETS`：`N`是直方图收集的桶数。如果未指定，将使用`fe.conf`中的默认值。

- PROPERTIES：自定义参数。如果未指定`PROPERTIES`，则使用`fe.conf`中的默认设置。

| **属性**                 | **类型** | **默认值** | **描述**                                              |
| ------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows  | 整型      | 200000            | 要收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。 |
| histogram_buckets_size         | 长整型     | 64                | 直方图的默认桶数。                   |
| histogram_mcv_size             | 整型      | 100               | 直方图的最常见值（MCV）数量。      |
| histogram_sample_ratio         | 浮点型    | 0.1               | 直方图的采样率。                          |
| histogram_max_sample_row_count | 长整型     | 10000000          | 直方图收集的最大行数。       |

直方图收集的行数由多个参数控制。它是`statistic_sample_collect_rows`和表行数*`histogram_sample_ratio`之间的较大值。该数字不能超过`histogram_max_sample_row_count`指定的值。如果超过该值，`histogram_max_sample_row_count`将优先。

可以通过`SHOW ANALYZE STATUS`输出中的`Properties`列查看实际使用的属性。

示例

```SQL
-- 使用默认设置在v1上手动收集直方图。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 在v1和v2上手动收集直方图，使用32个桶，32个MCV和50%的采样率。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

### 自定义收集

#### 自定义自动收集任务

您可以使用CREATE ANALYZE语句自定义自动收集任务。

在创建自定义自动收集任务之前，必须禁用自动全量收集（`enable_collect_full_statistic = false`）。否则，自定义任务将无法生效。

```SQL
-- 自动收集所有数据库的统计信息。
CREATE ANALYZE [FULL|SAMPLE] ALL [PROPERTIES (property [,property])]

-- 自动收集数据库中所有表的统计信息。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
[PROPERTIES (property [,property])]

-- 自动收集表中指定列的统计信息。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[PROPERTIES (property [,property])]
```

参数说明：

- 收集类型
  - FULL：表示全量收集。
  - SAMPLE：表示采样收集。
  - 如果未指定收集类型，则默认使用完整收集。

- `col_name`：要收集统计信息的列。用逗号（,）分隔多个列。如果未指定此参数，则收集整个表。

- `PROPERTIES`：自定义参数。如果未指定`PROPERTIES`，则使用`fe.conf`中的默认设置。

| **属性**                        | **类型** | **默认值** | **描述**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_auto_collect_ratio          | 浮点型    | 0.8               | 用于确定自动收集统计信息是否正常的阈值。如果统计健康度低于此阈值，则会触发自动收集。 |
| statistics_max_full_collect_data_size | 整型      | 100               | 自动收集的最大分区大小。单位：GB。如果分区超过此值，则放弃完整收集，改为执行采样收集。 |
| statistic_sample_collect_rows         | 整型      | 200000            | 要收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。 |
| statistic_exclude_pattern             | 字符串   | null              | 作业中需要排除的数据库或表的名称。您可以指定不收集统计信息的数据库和表。请注意，这是一个正则表达式模式，匹配内容为 `database.table`。 |
| statistic_auto_collect_interval       | 长整型   |  0      | 自动收集的时间间隔。单位：秒。默认情况下，StarRocks根据表的大小选择`statistic_auto_collect_small_table_interval`或`statistic_auto_collect_large_table_interval`作为收集间隔。如果在创建分析作业时指定了`statistic_auto_collect_interval`属性，则此设置优先于`statistic_auto_collect_small_table_interval`和`statistic_auto_collect_large_table_interval`。

示例

自动全量收集

```SQL
-- 自动收集所有数据库的完整统计信息。
CREATE ANALYZE ALL;

-- 自动收集数据库的完整统计信息。
CREATE ANALYZE DATABASE db_name;

-- 自动收集数据库中所有表的完整统计信息。
CREATE ANALYZE FULL DATABASE db_name;

-- 自动收集表中指定列的完整统计信息。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 

-- 自动收集所有数据库的统计信息，排除指定数据库'db_name'。
CREATE ANALYZE ALL PROPERTIES (
   "statistic_exclude_pattern" = "db_name\."
);
```

自动采样收集

```SQL
-- 使用默认设置自动收集数据库中所有表的统计信息。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 自动收集数据库中所有表的统计信息，排除指定表'db_name.tbl_name'。
CREATE ANALYZE SAMPLE DATABASE db_name PROPERTIES (
   "statistic_exclude_pattern" = "db_name\.tbl_name"
);

-- 自动收集表中指定列的统计信息，指定统计健康度和要收集的行数。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES (
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);

```

#### 查看自定义收集任务

```SQL
SHOW ANALYZE JOB [WHERE predicate]
```

可以使用WHERE子句筛选结果。该语句返回以下列。

| **列**   | **描述**                                              |
| ------------ | ------------------------------------------------------------ |
| 同上           | 收集任务的ID。                               |
| 数据库     | 数据库名称。                                           |
| 表        | 表名。                                              |
| 列      | 列名。                                            |
| 类型         | 统计信息的类型，包括`FULL`和`SAMPLE`。       |
| 计划     | 调度类型。自动任务的类型为`SCHEDULE`。 |
| 属性   | 自定义参数。                                           |
| 状态       | 任务状态，包括PENDING、RUNNING、SUCCESS和FAILED。 |
| 上次工作时间 | 上次收集的时间。                             |
| 原因       | 任务失败的原因。如果任务执行成功，则返回NULL。 |

示例

```SQL
-- 查看所有自定义收集任务。
SHOW ANALYZE JOB


-- 查看数据库`test`的自定义采集任务。
SHOW ANALYZE JOB WHERE `database` = 'test';
```

#### 删除自定义采集任务

```SQL
DROP ANALYZE <ID>
```

可以使用SHOW ANALYZE JOB语句获取任务ID。

示例

```SQL
DROP ANALYZE 266030;
```

## 查看采集任务状态

您可以通过运行SHOW ANALYZE STATUS语句来查看所有当前任务的状态。此语句不能用于查看自定义采集任务的状态。要查看自定义采集任务的状态，请使用SHOW ANALYZE JOB。

```SQL
SHOW ANALYZE STATUS [WHERE predicate];
```

您可以使用`LILE or WHERE`来筛选要返回的信息。

此语句返回以下列。

| **列表名称** | **描述**                                              |
| ------------- | ------------------------------------------------------------ |
| Id            | 采集任务的ID。                               |
| 数据库      | 数据库名称。                                           |
| 桌子         | 表名。                                              |
| 列       | 列名。                                            |
| 类型          | 统计量的类型，包括FULL、SAMPLE和HISTOGRAM。 |
| 附表      | 计划类型。表示`ONCE`手动，表示`SCHEDULE`自动。 |
| 地位        | 任务的状态。                                      |
| 开始时间     | 任务开始执行的时间。                     |
| 结束时间       | 任务执行结束的时间。                       |
| 性能    | 自定义参数。                                           |
| 原因        | 任务失败的原因。如果执行成功，则返回NULL。 |

## 查看统计信息

### 查看基本统计信息的元数据

```SQL
SHOW STATS META [WHERE predicate]
```

此语句返回以下列。

| **列** | **描述**                                              |
| ---------- | ------------------------------------------------------------ |
| 数据库   | 数据库名称。                                           |
| 桌子      | 表名。                                              |
| 列    | 列名。                                            |
| 类型       | 统计信息的类型。`FULL`表示完全收集，表示`SAMPLE`抽样收集。 |
| 更新时间 | 当前表的最新统计信息更新时间。     |
| 性能 | 自定义参数。                                           |
| 健康    | 统计信息的运行状况。                       |

### 查看直方图的元数据

```SQL
SHOW HISTOGRAM META [WHERE predicate]
```

此语句返回以下列。

| **列** | **描述**                                              |
| ---------- | ------------------------------------------------------------ |
| 数据库   | 数据库名称。                                           |
| 桌子      | 表名。                                              |
| 列     | 列。                                                 |
| 类型       | 统计信息的类型。该值`HISTOGRAM`用于直方图。 |
| 更新时间 | 当前表的最新统计信息更新时间。     |
| 性能 | 自定义参数。                                           |

## 删除统计信息

您可以删除不需要的统计信息。删除统计信息时，统计信息的数据和元数据以及过期缓存中的统计信息都会被删除。请注意，如果自动收集任务正在进行中，则可能会再次收集以前删除的统计信息。可用于`SHOW ANALYZE STATUS`查看收集任务的历史记录。

### 删除基本统计信息

```SQL
DROP STATS tbl_name
```

### 删除直方图

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name]
```

## 取消采集任务

您可以使用KILL ANALYZE语句取消**正在运行**的采集任务，包括手动任务和自定义任务。

```SQL
KILL ANALYZE <ID>
```

手动收集任务的任务ID可以从SHOW ANALYZE STATUS中获取。自定义收集任务的任务ID可以从SHOW ANALYZE SHOW ANALYZE JOB获取。

## 其他 FE配置项

| **FE配置项**        | **类型** | **默认值** | **描述**                                              |
| ------------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_collect_concurrency        | INT      | 3                 | 可以并行运行的手动收集任务的最大数量。该值默认为3，这意味着您最多可以并行运行三个手动收集任务。如果超过该值，传入任务将处于PENDING状态，等待调度。 |
| statistic_manager_sleep_time_sec     | LONG     | 60                | 计划元数据的时间间隔。单位：秒。系统根据此间隔执行以下操作：创建用于存储统计信息的表。删除已删除的统计信息。删除过期的统计信息。 |
| statistic_analyze_status_keep_second | LONG     | 259200            | 保留收集任务历史记录的持续时间。单位：秒。默认值：259200（3天）。 |

## 会话变量

`statistic_collect_parallel`：用于调整可在BE上运行的统计信息采集任务的并行度。默认值：1.您可以增加此值以加快收集任务。

## 收集Hive/Iceberg/Hudi表的统计信息

从v3.2.0开始，StarRocks支持收集Hive、Iceberg和Hudi表的统计信息。语法类似于收集StarRocks内部表。**但是，仅支持手动和自动完全收集。不支持采样收集和直方图收集。**收集的统计信息存储在`default_catalog`的`_statistics_`中的`external_column_statistics`表中。它们不存储在Hive Metastore中，也无法被其他搜索引擎共享。您可以查询`default_catalog._statistics_.external_column_statistics`表中的数据，验证是否已为Hive/Iceberg/Hudi表收集了统计信息。

以下是从`external_column_statistics`查询统计数据的示例。

```sql
SELECT * FROM _statistics_.external_column_statistics\G
*************************** 1. row ***************************
    table_uuid: hive_catalog.tn_test.ex_hive_tbl.1673596430
partition_name: 
   column_name: k1
  catalog_name: hive_catalog
       db_name: tn_test
    table_name: ex_hive_tbl
     row_count: 3
     data_size: 12
           ndv: NULL
    null_count: 0
           max: 3
           min: 2
   update_time: 2023-12-01 14:57:27.137000
```

### 限制

收集Hive、Iceberg、Hudi表的统计信息时，存在以下限制：

1. 仅支持收集Hive、Iceberg和Hudi表的统计信息。
2. 仅支持完整收集。不支持采样收集和直方图收集。
3. 为了让系统自动收集全量统计信息，您需要创建一个Analyze作业，这与收集StarRocks内部表的统计信息不同，StarRocks内部表的统计信息默认在后台进行。
4. 对于自动收集任务，您只能收集特定表的统计信息。不能收集数据库中所有表的统计信息，也不能收集外部目录中所有数据库的统计信息。
5. 对于自动收集任务，StarRocks可以检测Hive和Iceberg表中的数据是否更新，如果更新，则只收集数据更新的分区的统计信息。StarRocks无法感知Hudi表中的数据是否更新，只能进行周期性的全量收集。

以下示例发生在Hive外部目录下的数据库中。如果要从`default_catalog`收集Hive表的统计信息，请按以下格式引用该表`[catalog_name.][database_name.]<table_name>`。

### 手动收集

您可以按需创建Analyze作业，该作业在创建后立即运行。

#### 创建手动收集任务

语法：

```sql
ANALYZE [FULL] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES(property [,property])]
```

例：

```sql
ANALYZE TABLE ex_hive_tbl(k1);
+----------------------------------+---------+----------+----------+
| Table                            | Op      | Msg_type | Msg_text |
+----------------------------------+---------+----------+----------+
| hive_catalog.tn_test.ex_hive_tbl | analyze | status   | OK       |
+----------------------------------+---------+----------+----------+
```

#### 查看任务状态

语法：

```sql
SHOW ANALYZE STATUS [LIKE | WHERE predicate]
```

例：

```sql
SHOW ANALYZE STATUS where `table` = 'ex_hive_tbl';
+-------+----------------------+-------------+---------+------+----------+---------+---------------------+---------------------+------------+--------+
| Id    | Database             | Table       | Columns | Type | Schedule | Status  | StartTime           | EndTime             | Properties | Reason |
+-------+----------------------+-------------+---------+------+----------+---------+---------------------+---------------------+------------+--------+
| 16400 | hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | ONCE     | SUCCESS | 2023-12-04 16:31:42 | 2023-12-04 16:31:42 | {}         |        |
| 16465 | hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | ONCE     | SUCCESS | 2023-12-04 16:37:35 | 2023-12-04 16:37:35 | {}         |        |
| 16467 | hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | ONCE     | SUCCESS | 2023-12-04 16:37:46 | 2023-12-04 16:37:46 | {}         |        |
+-------+----------------------+-------------+---------+------+----------+---------+---------------------+---------------------+------------+--------+
```

#### 查看统计信息的元数据

语法：

```sql
SHOW STATS META [WHERE predicate]
```

例：

```sql
SHOW STATS META where `table` = 'ex_hive_tbl';
+----------------------+-------------+---------+------+---------------------+------------+---------+
| Database             | Table       | Columns | Type | UpdateTime          | Properties | Healthy |
+----------------------+-------------+---------+------+---------------------+------------+---------+
| hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | 2023-12-04 16:37:46 | {}         |         |
+----------------------+-------------+---------+------+---------------------+------------+---------+
```

#### 取消收集任务

取消正在运行的收集任务。

语法：

```sql
KILL ANALYZE <ID>
```

您可以在SHOW ANALYZE STATUS的输出中查看任务ID。

### 自动收集

为了使系统自动收集外部数据源中表的统计信息，您可以创建Analyze作业。StarRocks会自动检查是否以默认的5分钟检查间隔运行任务。对于Hive和Iceberg表，StarRocks仅在表中数据更新时执行采集任务。

但是，Hudi表中的数据变化是无法感知的，StarRocks会根据您设置的检查间隔和采集间隔定期进行统计。创建Analyze作业时，可以指定以下属性：

- statistic_collect_interval_sec

  自动采集过程中检查数据更新的时间间隔。单位：秒。默认值：5分钟。

- statistic_auto_collect_small_table_rows（v3.2及更高版本）

  阈值，用于判断外部数据源（Hive、Iceberg、Hudi）中的表在自动采集时是否为小表。默认值：10000000。

- statistic_auto_collect_small_table_interval

  收集小表统计信息的时间间隔。单位：秒。默认值：0。

- statistic_auto_collect_large_table_interval

  收集大型表统计信息的时间间隔。单位：秒。默认值：43200（12小时）。

自动收集线程按指定的时间间隔检查数据更新`statistic_collect_interval_sec`。如果表中的行数小于`statistic_auto_collect_small_table_rows`，则根据收集此类表的统计信息`statistic_auto_collect_small_table_interval`。

如果表中的行数超过`statistic_auto_collect_small_table_rows`，则根据收集此类表的统计信息`statistic_auto_collect_large_table_interval`。仅当`Last table update time + Collection interval > Current time`这样可以防止频繁执行大型表的分析任务。

#### 创建自动采集任务

语法：

```sql
CREATE ANALYZE TABLE tbl_name (col_name [,col_name])
[PROPERTIES (property [,property])]

```

例：

```sql
CREATE ANALYZE TABLE ex_hive_tbl (k1)
PROPERTIES ("statistic_auto_collect_interval" = "5");

Query OK, 0 rows affected (0.01 sec)
```

#### 查看自动采集任务的状态

与手动采集相同。

#### 查看统计元数据

与手动采集相同。

#### 查看自动采集任务

语法：

```sql
SHOW ANALYZE JOB [WHERE predicate]
```

例：

```sql
SHOW ANALYZE JOB WHERE `id` = '17204';

Empty set (0.00 sec)
```

#### 取消采集任务

与手动采集相同。

#### 删除统计信息

```sql
DROP STATS tbl_name
```

## 参考

- 要查询 FE 配置项，请运行 [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md)。

- 要修改 FE 配置项，请运行 [ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)。

