---
displayed_sidebar: "Chinese"
---

# 为CBO收集统计信息

此主题介绍了StarRocks成本优化器（CBO）的基本概念以及如何为CBO收集统计信息以选择最佳查询计划。从StarRocks 2.4开始引入直方图以收集准确的数据分布统计信息。自v3.2.0起，StarRocks支持从Hive、Iceberg和Hudi表中收集统计信息，减少了对其他元存储系统的依赖。语法类似于收集StarRocks内部表。

## 什么是CBO

CBO对查询优化至关重要。当SQL查询到达StarRocks时，它被解析为逻辑执行计划。CBO将逻辑计划重写和转换为多个物理执行计划。然后，CBO估算计划中每个操作符（如CPU、内存、网络和I/O）的执行成本，并选择具有最低成本的查询路径作为最终物理计划。

StarRocks CBO在StarRocks 1.16.0中推出，并从1.19版本开始默认启用。基于Cascades框架开发，StarRocks CBO根据各种统计信息估算成本。它能够在数万个执行计划中选择最低成本的执行计划，显著提高复杂查询的效率和性能。

统计信息对于CBO非常重要。它确定成本估算是否准确和有用。以下各节详细介绍了统计信息的类型、收集策略以及如何收集统计信息和查看统计信息。

## 统计信息的类型

StarRocks收集各种统计信息作为成本估算的输入。

### 基本统计信息

默认情况下，StarRocks周期性地收集表和列的以下基本统计信息：

- row_count：表中的行总数

- data_size：列的数据大小

- ndv：列的基数，即列中不同值的数量

- null_count：列中具有NULL值的数据量

- min：列中的最小值

- max：列中的最大值

完整的统计信息存储在`_statistics_.column_statistics`数据库的`column_statistics`表中。您可以查看此表以查询表格统计信息。以下是从此表查询统计数据的示例。

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

StarRocks 2.4引入了直方图来补充基本统计信息。直方图被认为是一种有效的数据表示方式。对于数据倾斜的表，直方图可以准确地反映数据分布。

StarRocks使用等高度直方图，它们构建在几个桶上。每个桶包含相等数量的数据。对于频繁查询并对选择性产生重大影响的数据值，StarRocks为它们分配单独的桶。更多的桶意味着更准确的估算，但也可能导致内存使用略微增加。您可以调整直方图收集任务的桶数和最常见值（MCV）。

**直方图适用于具有高度数据倾斜和频繁查询的列。如果表数据均匀分布，您无需创建直方图。直方图只能在数字、日期、日期时间或字符串类型的列上创建。**

目前，StarRocks仅支持手动收集直方图。直方图存储在`_statistics_.histogram_statistics`数据库的`histogram_statistics`表中。以下是从此表查询统计数据的示例：

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

表中的数据大小和数据分布不断变化。统计信息必须定期更新以表示这些数据变化。在创建统计信息收集任务之前，您必须选择最适合您业务需求的收集类型和方法。

StarRocks支持全量和抽样收集，可以自动执行和手动执行。默认情况下，StarRocks自动收集表的全量统计信息。它每5分钟检查一次数据更新。如果检测到数据更改，将自动触发数据收集。如果不想使用自动全量收集，您可以设置FE配置项`enable_collect_full_statistic`为`false`并定制一个收集任务。

| **收集类型** | **收集方法** | **描述**                                              | **优势和劣势**                               |
| ------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 全量收集     | 自动/手动      | 扫描整个表以收集统计信息。统计信息按分区收集。如果分区没有数据更改，将不从该分区收集数据，从而降低资源消耗。全量统计信息存储在`_statistics_.column_statistics`表中。 | 优势：统计信息准确，有助于CBO进行准确估算。劣势：消耗系统资源，速度较慢。从2.5版本开始，StarRocks允许您指定自动收集的周期，从而降低资源消耗。 |
| 抽样收集  | 自动/手动      | 从表的每个分区均匀抽取`N`行数据。统计信息按表收集。每列的基本统计信息存储为一条记录。根据抽样数据估计列的基数信息（ndv），这种方法不准确。抽样统计信息存储在`_statistics_.table_statistic_v1`表中。 | 优势：消耗的系统资源较少，速度快。劣势：统计信息不完整，可能影响成本估算的准确性。 |

## 收集统计信息

StarRocks提供了灵活的统计信息收集方法。您可以选择自动收集、手动收集或自定义收集，以适应您的业务场景。

### 自动全量收集

对于基本统计信息，StarRocks默认情况下自动全量收集表的基本统计信息，无需手动操作。对尚未收集统计信息的表，StarRocks将在调度周期内自动收集统计信息。对已收集统计信息的表，StarRocks会定期更新表中的行总数和已修改行数，并持久保存该信息以判断是否触发自动收集。

从2.4.5版本开始，StarRocks允许您为自动全量收集指定一个收集周期，以防止因自动全量收集而导致集群性能抖动。该周期由FE参数`statistic_auto_analyze_start_time`和`statistic_auto_analyze_end_time`指定。

触发自动收集的条件：

- 自上次统计信息收集以来表数据发生更改。

- 表统计信息的健康状况低于已配置的阈值（`statistic_auto_collect_ratio`）。

> 统计信息健康状况计算公式：1 - 自上次统计信息收集以来增加的行数/最小分区中的总行数

- 分区数据已被修改。未修改数据的分区将不再次收集。

- 收集时间落在已配置的收集周期范围内。（默认收集周期为全天。）

自动全量收集默认情况下启用，并由系统使用默认设置运行。

以下表格描述了默认设置。如果需修改这些设置，运行**ADMIN SET CONFIG**命令。

| **FE配置项**         | **类型** | **默认值** | **描述**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| enable_statistic_collect              | 布尔型  | TRUE              | 是否收集统计信息。默认情况下此开关打开。 |
| enable_collect_full_statistic         | 布尔型  | TRUE              | 是否启用自动全量收集。默认情况下此开关打开。 |
| statistic_collect_interval_sec        | 长整型     | 300               | 自动收集中检查数据更新的间隔。单位：秒。 |
| statistic_auto_analyze_start_time | 字符串      | 00:00:00   | 自动收集的开始时间。取值范围：`00:00:00` - `23:59:59`。 |
| statistic_auto_analyze_end_time | 字符串      | 23:59:59  | 自动收集的结束时间。取值范围：`00:00:00` - `23:59:59`。 |
| statistic_auto_collect_small_table_size     | 长整型    | 5368709120   | 用于确定表是否为小表的全量自动收集阈值。大小大于此值的表被视为大表，大小小于或等于此值的表被视为小表。单位：字节。默认值：5368709120（5GB）。                         |
| statistic_auto_collect_small_table_interval | 长整型    | 0         | 自动全量收集小表统计信息的间隔。单位：秒。                              |
| statistic_auto_collect_large_table_interval | LONG    | 43200        | 自动收集大表完整统计信息的间隔。单位：秒。默认值：43200（12小时）。                               |
| statistic_auto_collect_ratio          | FLOAT    | 0.8               | 用于确定自动收集统计信息是否健康的阈值。如果统计信息健康度低于此阈值，则会触发自动收集。 |
| statistic_max_full_collect_data_size | LONG      | 107374182400      | 自动收集的最大分区大小。单位：字节。默认值：107374182400（100 GB）。如果一个分区超过此值，则会放弃完整收集，改为执行抽样收集。 |
| statistic_collect_max_row_count_per_query | INT  | 5000000000        | 单个分析任务要查询的最大行数。如果超过此值，将对分析任务进行拆分查询。 |

对于绝大部分统计信息收集，您可以依赖自动作业，但如果您有特定的统计需求，您可以通过执行ANALYZE TABLE语句手动创建任务，或通过执行CREATE ANALYZE语句定制自动任务。

### 手动收集

您可以使用ANALYZE TABLE来创建手动收集任务。默认情况下，手动收集是同步操作。您也可以将其设置为异步操作。在异步模式中，运行ANALYZE TABLE后，系统会立即返回该语句是否成功。但是，收集任务将在后台运行，您无需等待结果。您可以通过运行SHOW ANALYZE STATUS来检查任务状态。异步收集适用于数据量大的表，而同步收集适用于数据量小的表。**手动收集任务仅在创建后运行一次。您无需删除手动收集任务。**

#### 手动收集基本统计信息

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES (property [,property])]
```

参数说明：

- 收集类型
  - FULL：表示完整收集。
  - SAMPLE：表示抽样收集。
  - 如果未指定收集类型，默认使用完整收集。

- `col_name`：要收集统计信息的列。使用逗号（`,`）分隔多个列。如果未指定此参数，则收集整个表。

- [WITH SYNC | ASYNC MODE]：是否将手动收集任务运行在同步或异步模式。如果未指定此参数，默认使用同步收集。

- `PROPERTIES`：自定义参数。如果未指定`PROPERTIES`，则使用`fe.conf`文件中的默认设置。实际使用的属性可以通过SHOW ANALYZE STATUS输出的`Properties`列进行查看。

| **PROPERTIES**                | **类型** | **默认值** | **描述**                         |
| ----------------------------- | -------- | ---------- | ----------------------------- |
| statistic_sample_collect_rows | INT      | 200000     | 抽样收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。 |

示例

手动完整收集

```SQL
-- 使用默认设置手动完整收集表的统计信息。
ANALYZE TABLE tbl_name;

-- 使用默认设置手动完整收集表的统计信息。
ANALYZE FULL TABLE tbl_name;

-- 使用默认设置手动收集表中指定列的统计信息。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

手动抽样收集

```SQL
-- 使用默认设置手动抽样收集表的部分统计信息。
ANALYZE SAMPLE TABLE tbl_name;

-- 使用指定行数手动收集表中指定列的统计信息。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES (
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

- `col_name`：要从中收集统计信息的列。使用逗号（`,`）分隔多个列。如果未指定此参数，则收集整个表。对于直方图，此参数是必需的。

- [WITH SYNC | ASYNC MODE]：是否将手动收集任务运行在同步或异步模式。如果未指定此参数，默认使用同步收集。

- `WITH N BUCKETS`：`N`是直方图收集的桶数。如果未指定，将使用`fe.conf`中的默认值。

- `PROPERTIES`：自定义参数。如果未指定`PROPERTIES`，则使用`fe.conf`中的默认设置。

| **PROPERTIES**                 | **类型** | **默认值** | **描述**                         |
| ------------------------------ | -------- | ---------- | ---------------------------- |
| statistic_sample_collect_rows  | INT      | 200000     | 要收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。 |
| histogram_buckets_size         | LONG     | 64         | 直方图的默认桶数。                   |
| histogram_mcv_size             | INT      | 100        | 直方图的最常见值（MCV）数。        |
| histogram_sample_ratio         | FLOAT    | 0.1        | 直方图的抽样比率。                  |
| histogram_max_sample_row_count | LONG     | 10000000   | 要收集的直方图的最大行数。             |

直方图的收集行数由多个参数控制。它是`statistic_sample_collect_rows`和表行数*`histogram_sample_ratio`之间的较大值。该数字不能超过由`histogram_max_sample_row_count`指定的值。如果超过该值，则以`histogram_max_sample_row_count`为准。

实际使用的属性可以通过SHOW ANALYZE STATUS输出的`Properties`列进行查看。

示例

```SQL
-- 使用默认设置手动在v1上收集直方图。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 在v1和v2上手动收集直方图，使用32个桶，32个MCV和50%的抽样比率。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

### 自定义收集

#### 自定义自动收集任务

您可以使用CREATE ANALYZE语句来定制自动收集任务。

在创建自定义自动收集任务之前，您必须禁用自动完整收集（`enable_collect_full_statistic=false`）。否则，自定义任务将无法生效。

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
  - FULL：表示完整收集。
  - SAMPLE：表示抽样收集。
  - 如果未指定收集类型，默认使用完整收集。

- `col_name`：要收集统计信息的列。使用逗号（`,`）分隔多个列。如果未指定此参数，则收集整个表。

- `PROPERTIES`：自定义参数。如果未指定`PROPERTIES`，则使用`fe.conf`中的默认设置。

| **PROPERTIES**                        | **类型** | **默认值** | **描述**                               |
| ------------------------------------- | -------- | ---------- | ----------------------------------- |
| statistic_auto_collect_ratio          | FLOAT    | 0.8        | 用于确定自动收集统计信息是否健康的阈值。如果统计健康度低于此阈值，则会触发自动收集。 |
| statistics_max_full_collect_data_size | INT      | 100        | 自动收集的最大分区大小。单位：GB。如果一个分区超过此值，则会放弃完整收集，改为执行抽样收集。 |
| statistic_sample_collect_rows         | INT      | 200000     | 要收集的最小行数。如果参数值超过表中的实际行数，则执行完整收集。 |
| statistic_exclude_pattern             | 字符串   | null       | 需要在作业中排除的数据库或表的名称。您可以指定不收集作业中统计信息的数据库和表。请注意，这是一个正则表达式模式，匹配的内容是`database.table`。 |
| statistic_auto_collect_interval           | LONG   |  0      | 自动收集统计信息的间隔。单位：秒。默认情况下，StarRocks根据表大小选择`statistic_auto_collect_small_table_interval`或`statistic_auto_collect_large_table_interval`作为收集间隔。如果在创建分析作业时指定了`statistic_auto_collect_interval`属性，则此设置优先于`statistic_auto_collect_small_table_interval`和`statistic_auto_collect_large_table_interval`。 |

示例

自动全量收集

```SQL
-- 自动收集所有数据库的完整统计信息。
CREATE ANALYZE ALL;

-- 自动收集数据库的完整统计信息。
CREATE ANALYZE DATABASE db_name;

-- 自动收集数据库中所有表的完整统计信息。
CREATE ANALYZE FULL DATABASE db_name;

-- 自动收集表中指定列的统计信息。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 

-- 自动收集所有数据库的统计信息，但不包括指定的数据库'db_name'。
CREATE ANALYZE ALL PROPERTIES (
   "statistic_exclude_pattern" = "db_name\."
);
```

自动抽样收集

```SQL
-- 使用默认设置自动收集数据库中所有表的统计信息。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 自动收集数据库中所有表的统计信息，但不包括指定的表'db_name.tbl_name'。
CREATE ANALYZE SAMPLE DATABASE db_name PROPERTIES (
   "statistic_exclude_pattern" = "db_name\.tbl_name"
);

-- 在表中自动收集指定列的统计信息，并指定统计信息健康度和要收集的行数。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES (
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);

```

#### 查看自定义收集任务

```SQL
SHOW ANALYZE JOB [WHERE predicate]
```

您可以使用`WHERE`子句过滤结果。该语句返回以下列。

| **列名**      | **描述**                         |
| ------------ | -------------------------------- |
| Id           | 收集任务的ID。                   |
| Database     | 数据库名称。                     |
| Table        | 表名称。                         |
| Columns      | 列名称。                         |
| Type         | 统计类型，包括`FULL`和`SAMPLE`。 |
| Schedule     | 调度类型。自动任务的类型是`SCHEDULE`。 |
| Properties   | 自定义参数。                     |
| Status       | 任务状态，包括PENDING、RUNNING、SUCCESS和FAILED。 |
| LastWorkTime | 上次收集时间。                   |
| Reason       | 任务失败的原因。如果任务执行成功，则返回NULL。 |

示例

```SQL
-- 查看所有自定义收集任务。
SHOW ANALYZE JOB

-- 查看数据库`test`的自定义收集任务。
SHOW ANALYZE JOB where `database` = 'test';
```

#### 删除自定义收集任务

```SQL
DROP ANALYZE <ID>
```

任务ID可通过使用`SHOW ANALYZE JOB`语句获取。

示例

```SQL
DROP ANALYZE 266030;
```

## 查看收集任务状态

您可以通过运行`SHOW ANALYZE STATUS`语句来查看所有当前任务的状态。此语句不能用于查看自定义收集任务的状态。要查看自定义收集任务的状态，请使用`SHOW ANALYZE JOB`。

```SQL
SHOW ANALYZE STATUS [WHERE predicate];
```

您可以使用`LIKE`或`WHERE`来过滤要返回的信息。

此语句返回以下列。

| **列表名称** | **描述**                         |
| ------------ | -------------------------------- |
| Id           | 收集任务的ID。                   |
| Database     | 数据库名称。                     |
| Table        | 表名称。                         |
| Columns      | 列名称。                         |
| Type         | 统计类型，包括FULL、SAMPLE和HISTOGRAM。 |
| Schedule     | 调度类型。`ONCE`表示手动，`SCHEDULE`表示自动。 |
| Status       | 任务状态。                       |
| StartTime    | 任务开始执行时的时间。            |
| EndTime      | 任务执行结束时的时间。            |
| Properties   | 自定义参数。                     |
| Reason       | 任务失败的原因。如果执行成功，则返回NULL。 |

## 查看统计信息

### 查看基本统计信息元数据

```SQL
SHOW STATS META [WHERE predicate]
```

此语句返回以下列。

| **列名**   | **描述**                         |
| ---------- | -------------------------------- |
| Database   | 数据库名称。                     |
| Table      | 表名称。                         |
| Columns    | 列名称。                         |
| Type       | 统计类型。`FULL`表示完整收集，`SAMPLE`表示抽样收集。 |
| UpdateTime | 当前表的最新统计更新时间。       |
| Properties | 自定义参数。                     |
| Healthy    | 统计信息的健康状况。             |

### 查看直方图元数据

```SQL
SHOW HISTOGRAM META [WHERE predicate]
```

此语句返回以下列。

| **列名** | **描述**                         |
| -------- | -------------------------------- |
| Database | 数据库名称。                     |
| Table    | 表名称。                         |
| Column   | 列名称。                         |
| Type     | 统计类型。直方图的值为`HISTOGRAM`。 |
| UpdateTime | 当前表的最新统计更新时间。     |
| Properties | 自定义参数。                   |

## 删除统计信息

您可以删除不需要的统计信息。在删除统计信息时，将同时删除统计数据和元数据，以及过期缓存中的统计信息。请注意，如果自动收集任务正在进行中，则先前删除的统计信息可能会被再次收集。您可以使用`SHOW ANALYZE STATUS`查看收集任务的历史记录。

### 删除基本统计信息

```SQL
DROP STATS tbl_name
```

### 删除直方图

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name]
```

## 取消收集任务

您可以使用`KILL ANALYZE`语句来取消**正在运行**的收集任务，包括手动任务和自定义任务。

```SQL
KILL ANALYZE <ID>
```

手动收集任务的任务ID可以从`SHOW ANALYZE STATUS`获取。自定义收集任务的任务ID可以从`SHOW ANALYZE SHOW ANALYZE JOB`获取。

## 其他FE配置项

| **FE配置项**                            | **类型** | **默认值** | **描述**                         |
| --------------------------------------- | -------- | ---------- | -------------------------------- |
| statistic_collect_concurrency           | INT      | 3          | 能够并行运行的手动收集任务的最大数量。默认值为3，这意味着最多可以并行运行三个手动收集任务。如果超出该值，进入的任务将处于PENDING状态，等待调度。 |
| statistic_manager_sleep_time_sec       | LONG     | 60         | 元数据进行调度的间隔。单位：秒。系统基于此间隔执行以下操作：创建用于存储统计信息的表。删除已删除的统计信息。删除过期的统计信息。 |
| statistic_analyze_status_keep_second    | LONG     | 259200     | 保留收集任务历史记录的时长。单位：秒。默认值：259200（3天）。 |

## 会话变量

`statistic_collect_parallel`：用于调整BE上可以运行的统计收集任务的并行度。默认值：1。您可以增加此值以加快收集任务的速度。

## 收集Hive/Iceberg/Hudi表的统计信息

自v3.2.0以来，StarRocks支持收集Hive、Iceberg和Hudi表的统计信息。语法与收集StarRocks内部表的统计信息类似。**但是，仅支持手动和自动完整收集。不支持抽样收集或直方图收集。**收集的统计信息存储在`default_catalog`中`_statistics_`的`external_column_statistics`表中。它们不存储在Hive Metastore中，也不能被其他搜索引擎共享。您可以查询`default_catalog._statistics_.external_column_statistics`表中的数据，以验证是否为Hive/Iceberg/Hudi表收集了统计信息。

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

对于Hive、Iceberg、Hudi表的统计收集，应用以下限制：

1. 仅支持全量收集。不支持采样收集和直方图收集。
2. 只能收集Hive、Iceberg和Hudi表的统计信息。
3. 对于自动收集任务，只能收集特定表的统计信息。无法收集数据库中所有表的统计信息或外部目录中所有数据库的统计信息。
4. 对于自动收集任务，StarRocks可以检测Hive和Iceberg表中的数据是否进行了更新，如果有更新，则收集只有数据更新的分区的统计信息。StarRocks无法感知Hudi表中的数据是否已更新，并且只能执行定期的全量收集。

以下示例发生在Hive外部目录下的数据库中。如果要收集来自`default_catalog`的Hive表的统计信息，请使用`[catalog_name.][database_name.]<table_name>`格式引用该表。

### 手动收集

#### 创建手动收集任务

语法：

```sql
ANALYZE [FULL] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES(property [,property])]
```

示例：

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

示例：

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

示例：

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

创建自动收集任务时，StarRocks会自动检查是否在默认的5分钟检查间隔内运行任务。StarRocks仅在Hive和Iceberg表中的数据更新时运行收集任务。然而，无法感知Hudi表中的数据变化，而StarRocks会根据用户指定的检查间隔和收集间隔定期收集统计信息。您可以使用以下FE参数：

- statistic_collect_interval_sec

  自动收集期间检查数据更新的间隔。单位：秒。默认值：5分钟。

- statistic_auto_collect_small_table_rows（v3.2及更高版本）

  在自动收集期间确定外部数据源（Hive、Iceberg、Hudi）中表是否为小表的阈值。默认值：10000000。

- statistic_auto_collect_small_table_interval

  收集小表统计信息的间隔。单位：秒。默认值：0。

- statistic_auto_collect_large_table_interval

  收集大表统计信息的间隔。单位：秒。默认值：43200（12小时）。

自动收集线程会根据`statistic_collect_interval_sec`指定的间隔检查数据更新。如果表中的行数少于`statistic_auto_collect_small_table_rows`，它会根据`statistic_auto_collect_small_table_interval`收集这些表的统计信息。

如果表中的行数超过`statistic_auto_collect_small_table_rows`，它会根据`statistic_auto_collect_large_table_interval`收集这些表的统计信息。只有在`最后表更新时间 + 收集间隔 > 当前时间`时，才会更新大表的统计信息。这可以防止频繁针对大表进行分析任务。

#### 创建自动收集任务

语法：

```sql
CREATE ANALYZE TABLE tbl_name (col_name [,col_name])
[PROPERTIES (property [,property])]
```

示例：

```sql
CREATE ANALYZE TABLE ex_hive_tbl (k1)
PROPERTIES ("statistic_auto_collect_interval" = "5");

Query OK, 0 rows affected (0.01 sec)
```

#### 查看自动收集任务的状态

与手动收集相同。

#### 查看统计信息的元数据

与手动收集相同。

#### 查看自动收集任务

语法：

```sql
SHOW ANALYZE JOB [WHERE predicate]
```

示例：

```sql
SHOW ANALYZE JOB WHERE `id` = '17204';

Empty set (0.00 sec)
```

#### 取消收集任务

与手动收集相同。

### 删除统计信息

```sql
DROP STATS tbl_name
```

## 参考

- 要查询FE配置项，请运行[ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md)。

- 要修改FE配置项，请运行[ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)。