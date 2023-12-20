---
displayed_sidebar: English
---

# 收集 CBO 的统计数据

本文介绍了 StarRocks 基于成本的优化器（CBO）的基本概念，以及如何收集 CBO 的统计信息以选择最优的查询计划。StarRocks 2.4 引入了直方图来收集准确的数据分布统计信息。

自 v3.2.0 起，StarRocks 支持从 Hive、Iceberg 和 Hudi 表收集统计信息，减少了对其他元数据存储系统的依赖。语法与收集 StarRocks 内部表类似。

## 什么是 CBO

CBO 对查询优化至关重要。SQL 查询到达 StarRocks 后，会被解析为逻辑执行计划。CBO 重写并转换逻辑计划为多个物理执行计划。CBO 接着估计计划中每个算子的执行成本（如 CPU、内存、网络和 I/O），并选择成本最低的查询路径作为最终的物理计划。

StarRocks CBO 自 StarRocks 1.16.0 版本启动，并从 1.19 版本起默认启用。基于 Cascades 框架开发的 StarRocks CBO，根据各种统计信息估算成本。它能够在数以万计的执行计划中选择成本最低的执行计划，显著提高复杂查询的效率和性能。

统计数据对 CBO 来说非常重要。它们决定了成本估算是否准确和有用。以下部分将详细介绍统计信息的类型、收集策略，以及如何收集和查看统计信息。

## 统计信息的类型

StarRocks 收集多种统计数据作为成本估算的输入。

### 基础统计

默认情况下，StarRocks 定期收集表和列的以下基础统计信息：

- row_count：表中的总行数
- data_size：列的数据大小
- ndv：列的基数，即列中不同值的数量
- null_count：列中含 NULL 值的数据量
- min：列中的最小值
- max：列中的最大值

完整的统计信息存储在 `_statistics_` 数据库的 `column_statistics` 表中。您可以查询此表以获取表的统计信息。以下是从此表查询统计数据的示例。

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

StarRocks 2.4 引入了直方图来补充基础统计。直方图被认为是数据表示的有效方式。对于数据分布不均的表，直方图可以准确反映数据分布。

StarRocks 使用等高直方图，这些直方图基于多个桶构建。每个桶包含相等数量的数据。对于查询频繁且对选择性有重大影响的数据值，StarRocks 会为它们分配单独的桶。桶越多，估算越准确，但也可能略微增加内存使用。您可以调整直方图收集任务的桶数和最常见值（MCVs）。

**直方图适用于数据分布极不均匀且查询频繁的列。如果您的表数据分布均匀，那么不需要创建直方图。直方图只能在数值、DATE、DATETIME 或字符串类型的列上创建。**

目前，StarRocks 仅支持手动收集直方图。直方图存储在 `_statistics_` 数据库的 `histogram_statistics` 表中。以下是从此表查询统计数据的示例：

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

表中的数据大小和分布不断变化。统计数据必须定期更新，以反映数据变化。在创建统计收集任务之前，您必须选择最适合您业务需求的收集类型和方法。

StarRocks 支持全量收集和采样收集，两者都可以自动或手动执行。默认情况下，StarRocks 自动收集表的全量统计信息。它每 5 分钟检查一次数据更新。如果检测到数据变化，则会自动触发数据收集。如果您不希望使用自动全量收集，可以将 FE 配置项 `enable_collect_full_statistic` 设置为 `false` 并自定义收集任务。

|**收集类型**|**收集方法**|**描述**|**优缺点**|
|---|---|---|---|
|全量收集|自动/手动|扫描全表以收集统计信息。统计信息按分区收集。如果某个分区没有数据变化，则不会从该分区收集数据，减少资源消耗。全量统计信息存储在 `_statistics_.column_statistics` 表中。|优点：统计信息准确，有助于 CBO 做出准确估算。缺点：消耗系统资源且速度慢。从 2.5 版本起，StarRocks 允许您指定自动收集周期，以减少资源消耗。|
|采样收集|自动/手动|从表的每个分区均匀抽取 `N` 行数据。按表收集统计信息。每列的基础统计信息存储为一条记录。列的基数信息（ndv）基于采样数据估算，不够准确。采样统计信息存储在 `_statistics_.table_statistic_v1` 表中。|优点：消耗较少系统资源，速度快。缺点：统计信息不完整，可能影响成本估算的准确性。|

## 收集统计数据

StarRocks 提供灵活的统计收集方法。您可以选择自动、手动或自定义收集，以适应您的业务场景。

### 自动收集

对于基础统计，StarRocks 默认自动收集表的全量统计信息，无需手动操作。对于尚未收集统计信息的表，StarRocks 会在调度周期内自动收集统计信息。对于已收集统计信息的表，StarRocks 会更新表中的总行数和修改行数，并定期持久化这些信息，以判断是否触发自动收集。

从 2.4.5 版本起，StarRocks 允许您指定自动全量收集的周期，防止自动全量收集引起的集群性能波动。该周期由 FE 参数 `statistic_auto_analyze_start_time` 和 `statistic_auto_analyze_end_time` 指定。

触发自动收集的条件包括：

- 自上次统计收集以来，表数据已发生变化。
- 收集时间在配置的收集周期范围内。（默认收集周期为全天。）
- 上一次收集作业的更新时间早于分区的最新更新时间。
- 表统计信息的健康度低于指定阈值（`statistic_auto_collect_ratio`）。

> 统计健康度计算公式：
> 如果更新数据的分区数小于 10，则公式为 `1 - (自上次收集以来更新的行数/总行数)`。
> 如果更新数据的分区数大于或等于 10，则公式为 `1 - MIN(自上次收集以来更新的行数/总行数, 自上次收集以来更新的分区数/总分区数)`。

此外，StarRocks 允许您根据表的大小和更新频率配置收集策略：

- 对于数据量较小的表，**统计信息实时收集，无限制，即使表数据频繁更新。`statistic_auto_collect_small_table_size` 参数用于判断表是小表还是大表。您还可以使用 `statistic_auto_collect_small_table_interval` 配置小表的收集间隔。

- 对于数据量较大的表，有以下限制：

  - 默认收集间隔不少于 12 小时，可使用 `statistic_auto_collect_large_table_interval` 配置。
  - 当满足收集间隔且统计健康度低于自动采样收集阈值（`statistic_auto_collect_sample_threshold`）时，触发采样收集。
  - 当满足收集间隔且统计健康度高于自动采样收集阈值（`statistic_auto_collect_sample_threshold`）且低于自动收集阈值（`statistic_auto_collect_ratio`）时，触发全量收集。
  - 当收集数据的最大分区大小（`statistic_max_full_collect_data_size`）超过 100 GB 时，触发采样收集。
  - 只收集上一次收集任务之后更新时间的分区的统计信息。未发生数据变化的分区不收集统计信息。

:::tip

表数据变化后，手动触发该表的采样收集任务会使采样收集任务的更新时间晚于数据更新时间，在该调度周期内不会触发该表的自动全量收集。:::

自动全量收集默认启用，并由系统使用默认设置运行。

下表描述了默认设置。如果需要修改它们，请运行 **ADMIN SET CONFIG** 命令。

|**FE 配置项**|**类型**|**默认值**|**描述**|
|---|---|---|---|
|enable_statistic_collect|BOOLEAN|TRUE|是否收集统计信息。此开关默认开启。|
|enable_collect_full_statistic|BOOLEAN|TRUE|是否启用自动全量收集。此开关默认开启。|
|statistic_collect_interval_sec|LONG|300|自动收集时检查数据更新的间隔。单位：秒。|
|statistic_auto_analyze_start_time|STRING|00:00:00|自动收集的开始时间。取值范围：`00:00:00` - `23:59:59`。|
|statistic_auto_analyze_end_time|STRING|23:59:59|自动收集的结束时间。取值范围：`00:00:00` - `23:59:59`。|
|statistic_auto_collect_small_table_size|LONG|5368709120|自动全量收集时判断表是否为小表的阈值。大于此值的表被视为大表，小于或等于此值的表被视为小表。单位：字节。默认值：5368709120（5 GB）。|
```
```SQL
DROP ANALYZE 266030;
```

## 查看采集任务状态

您可以通过运行 **SHOW ANALYZE STATUS** 语句查看所有当前任务的状态。此语句不能用于查看自定义采集任务的状态。要查看自定义采集任务的状态，请使用 **SHOW ANALYZE JOB**。

```SQL
SHOW ANALYZE STATUS [WHERE predicate];
```

您可以使用 `LIKE` 或 `WHERE` 来过滤返回的信息。

此语句返回以下列。

|**列名**|**描述**|
|---|---|
|Id|采集任务的 ID。|
|数据库|数据库名称。|
|表|表名称。|
|列|列名称。|
|类型|统计数据的类型，包括 FULL、SAMPLE 和 HISTOGRAM。|
|计划|调度类型。`ONCE` 表示手动，`SCHEDULE` 表示自动。|
|状态|任务状态。|
|开始时间|任务开始执行的时间。|
|结束时间|任务执行结束的时间。|
|属性|自定义参数。|
|原因|任务失败的原因。如果执行成功，则返回 NULL。|

## 查看统计信息

### 查看基本统计信息的元数据

```SQL
SHOW STATS META [WHERE predicate]
```

此语句返回以下列。

|**列名**|**描述**|
|---|---|
|数据库|数据库名称。|
|表|表名称。|
|列|列名称。|
|类型|统计数据的类型。`FULL` 表示全量采集，`SAMPLE` 表示采样采集。|
|更新时间|当前表的最新统计更新时间。|
|属性|自定义参数。|
|健康状况|统计信息的健康状况。|

### 查看直方图的元数据

```SQL
SHOW HISTOGRAM META [WHERE predicate]
```

此语句返回以下列。

|**列名**|**描述**|
|---|---|
|数据库|数据库名称。|
|表|表名称。|
|列|列。|
|类型|统计数据的类型。对于直方图，值为 `HISTOGRAM`。|
|更新时间|当前表的最新统计更新时间。|
|属性|自定义参数。|

## 删除统计信息

您可以删除不需要的统计信息。删除统计信息时，将删除统计数据和元数据，以及过期缓存中的统计信息。请注意，如果正在进行自动采集任务，则可能会再次收集先前删除的统计信息。您可以使用 `SHOW ANALYZE STATUS` 查看采集任务的历史记录。

### 删除基本统计信息

```SQL
DROP STATS tbl_name
```

### 删除直方图

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name]
```

## 取消采集任务

您可以使用 **KILL ANALYZE** 语句取消**正在运行**的采集任务，包括手动和自定义任务。

```SQL
KILL ANALYZE <ID>
```

手动采集任务的任务 ID 可以从 **SHOW ANALYZE STATUS** 获得。自定义采集任务的任务 ID 可以从 **SHOW ANALYZE JOB** 获得。

## 其他 FE 配置项

|**FE 配置项**|**类型**|**默认值**|**描述**|
|---|---|---|---|
|statistic_collect_concurrency|INT|3|可以并行运行的手动采集任务的最大数量。默认值为 3，这意味着您最多可以并行运行三个手动采集任务。如果超过此值，传入的任务将处于 PENDING 状态，等待调度。|
|statistic_manager_sleep_time_sec|LONG|60|元数据调度的间隔。单位：秒。系统根据此间隔执行以下操作：创建用于存储统计信息的表。删除已删除的统计信息。删除过期的统计信息。|
|statistic_analyze_status_keep_second|LONG|259200|保留采集任务历史记录的时长。单位：秒。默认值：259200（3天）。|

## 会话变量

`statistic_collect_parallel`：用于调整可以在 BE 上并行运行的统计信息采集任务的并行度。默认值：1。您可以增加此值以加快采集任务的速度。

## 收集 Hive/Iceberg/Hudi 表的统计信息

自 v3.2.0 起，StarRocks 支持收集 Hive、Iceberg 和 Hudi 表的统计信息。语法与收集 StarRocks 内部表类似。**但是，仅支持手动和自动全量采集。不支持采样采集和直方图采集。** 收集的统计信息存储在 `_statistics_` 的 `external_column_statistics` 表中，位于 `default_catalog`。它们不存储在 Hive Metastore 中，也不能被其他搜索引擎共享。您可以从 `default_catalog._statistics_.external_column_statistics` 表查询数据，以验证是否为 Hive/Iceberg/Hudi 表收集了统计信息。

以下是从 `external_column_statistics` 查询统计数据的示例。

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

收集 Hive、Iceberg、Hudi 表的统计信息时适用以下限制：

1. 您只能收集 Hive、Iceberg 和 Hudi 表的统计信息。
2. 仅支持全量采集。不支持采样采集和直方图采集。
3. 要自动收集全量统计信息，您必须创建一个 Analyze 作业，这与收集 StarRocks 内部表的统计信息不同，在后者中系统默认在后台执行此操作。
4. 对于自动采集任务，您只能收集特定表的统计信息。您不能收集数据库中所有表的统计信息，也不能收集外部目录中所有数据库的统计信息。
5. 对于自动采集任务，StarRocks 可以检测 Hive 和 Iceberg 表中的数据是否更新，如果是，则只收集数据更新的分区的统计信息。StarRocks 无法感知 Hudi 表中的数据是否更新，只能根据指定的周期进行全量采集。

以下示例发生在 Hive 外部目录下的数据库中。如果您想从 `default_catalog` 收集 Hive 表的统计信息，请按照 `[catalog_name.][database_name.]<table_name>` 格式引用表。

### 手动采集

您可以根据需要创建一个 Analyze 作业，并且在创建后作业会立即运行。

#### 创建手动采集任务

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

#### 查看统计信息元数据

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

#### 取消采集任务

取消正在运行的采集任务。

语法：

```sql
KILL ANALYZE <ID>
```

您可以在 **SHOW ANALYZE STATUS** 的输出中查看任务 ID。

### 自动采集

要让系统自动收集外部数据源中的表的统计信息，您可以创建一个 Analyze 作业。StarRocks 每隔默认的检查间隔（5 分钟）自动检查是否运行任务。对于 Hive 和 Iceberg 表，StarRocks 仅在表中的数据更新时运行采集任务。

然而，Hudi 表中的数据变化无法被感知，StarRocks 会根据您指定的检查间隔和采集间隔定期收集统计信息。创建 Analyze 作业时，您可以指定以下属性：

- statistic_collect_interval_sec

  自动采集期间检查数据更新的间隔。单位：秒。默认：5 分钟。

- statistic_auto_collect_small_table_rows（v3.2 及以后）

  自动采集期间判断外部数据源（Hive、Iceberg、Hudi）中的表是否为小表的阈值。默认：10000000。

- statistic_auto_collect_small_table_interval

  收集小表统计信息的间隔。单位：秒。默认：0。

- statistic_auto_collect_large_table_interval

  收集大表统计信息的间隔。单位：秒。默认：43200（12 小时）。

自动采集线程会在 `statistic_collect_interval_sec` 指定的间隔检查数据更新。如果表中的行数少于 `statistic_auto_collect_small_table_rows`，则根据 `statistic_auto_collect_small_table_interval` 收集此类表的统计信息。

如果表中的行数超过 `statistic_auto_collect_small_table_rows`，则根据 `statistic_auto_collect_large_table_interval` 收集此类表的统计信息。它仅在 `上次表更新时间 + 采集间隔 > 当前时间` 时更新大表的统计信息。这可以防止大表频繁进行分析任务。

#### 创建自动采集任务

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

#### 查看自动采集任务状态

与手动采集相同。

#### 查看统计信息元数据

与手动采集相同。

#### 查看自动采集任务

语法：

```sql
SHOW ANALYZE JOB [WHERE predicate]
```

示例：

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
## 查看自动采集任务的状态

句法：

```sql
SHOW ANALYZE JOB [WHERE predicate]
```

例子：

```sql
SHOW ANALYZE JOB WHERE `id` = '17204';

Empty set (0.00 sec)
```

#### 取消采集任务

与手动采集相同。

#### 删除统计数据

```sql
DROP STATS tbl_name
```

## 参考资料

- 要查询FE配置项，请运行 [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md)。

- 要修改FE配置项，请运行 [ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)。
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

示例：

```sql
SHOW ANALYZE JOB WHERE `id` = '17204';

Empty set (0.00 sec)
```

#### 取消采集任务

与手动采集相同。

#### 删除统计数据

```sql
DROP STATS tbl_name
```

## 参考资料

- 执行 [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md) 命令查询 FE 配置项。

- 要修改 FE 配置项，请运行 [ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)。