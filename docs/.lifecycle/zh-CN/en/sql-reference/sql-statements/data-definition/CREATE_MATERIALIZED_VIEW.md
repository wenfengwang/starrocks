---
displayed_sidebar: "中文"
---

# 创建物化视图

## 描述

创建一个物化视图。有关物化视图的使用信息，请参见[Synchronous materialized view](../../../using_starrocks/Materialized_view-single_table.md)和[Asynchronous materialized view](../../../using_starrocks/Materialized_view.md)。

> **注意**
>
> 只有在基表所在的数据库中具有CREATE MATERIALIZED VIEW权限的用户才能创建物化视图。

创建物化视图是一个异步操作。成功运行此命令表示已成功提交创建物化视图的任务。您可以通过[SHOW ALTER MATERIALIZED VIEW](../data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)命令查看数据库中同步物化视图的构建状态，也可以通过查询元数据视图[`tasks`](../../../reference/information_schema/tasks.md)和[`task_runs`](../../../reference/information_schema/task_runs.md)来查看异步物化视图的构建状态。

StarRocks从v2.4开始支持异步物化视图。与先前版本中的同步物化视图相比，异步物化视图的主要区别如下：

|                       | **单表聚合** | **多表关联** | **查询重写** | **刷新策略** | **基表** |
| --------------------- | ------------ | ------------ | ------------ | ------------ | -------- |
| **ASYNC MV** | 是 | 是 | 是 | <ul><li>异步刷新</li><li>手动刷新</li></ul> | 来自多个表：<ul><li>默认目录</li><li>外部目录（v2.5）</li><li>现有物化视图（v2.5）</li><li>现有视图（v3.1）</li></ul> |
| **SYNC MV（Rollup）** | 有限的聚合函数选择 | 否 | 是 | 在数据加载期间同步刷新 | 默认目录中的单表 |

## 同步物化视图

### 语法

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

方括号中的参数为可选项。

### 参数

**mv_name**（必需）

物化视图的名称。名称要求如下：

- 名称必须由字母（a-z或A-Z）、数字（0-9）或下划线(_)组成，并且只能以字母开头。
- 名称的长度不能超过64个字符。
- 名称区分大小写。

**COMMENT**（可选）

对物化视图的注释。请注意，`COMMENT`必须放在`mv_name`之后，否则将无法创建物化视图。

**query_statement**（必需）

创建物化视图的查询语句。其结果是物化视图中的数据。语法如下：

```SQL
SELECT select_expr[, select_expr ...]
[WHERE where_expr]
[GROUP BY column_name[, column_name ...]]
[ORDER BY column_name[, column_name ...]]
```

- select_expr（必需）

  查询语句中的所有列，即基表模式中的所有列。此参数支持以下值：

  - 简单列或聚合列，例如`SELECT a, abs(b), min(c) FROM table_a`，其中`a`、`b`和`c`是基表中的列名。如果未为物化视图指定列名，则StarRocks会自动为这些列分配名称。
  - 表达式，例如`SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a`，其中`a+1`、`b+2`和`c*c`是参考基表列的表达式，`x`、`y`和`z`是赋予物化视图中的列的别名。

  > **注意**
  >
  > - 必须至少指定一个列在`select_expr`中。
  > - 当创建具有聚合函数的同步物化视图时，必须指定GROUP BY子句，并在`select_expr`中至少指定一个GROUP BY列。
  > - 同步物化视图不支持JOIN和GROUP BY的HAVING子句。
  > - 从v3.1开始，每个同步物化视图可以支持基表的每列多个聚合函数，例如查询语句`select b, sum(a), min(a) from table group by b`。
  > - 从v3.1开始，同步物化视图支持SELECT和聚合函数的复杂表达式，例如查询语句`select b, sum(a + 1) as sum_a1, min(cast (a as bigint)) as min_a from table group by b`或`select abs(b) as col1, a + 1 as col2, cast(a as bigint) as col3 from table`。对用于同步物化视图的复杂表达式施加以下限制:
  >   - 每个复杂表达式必须具有别名，而不同的同步物化视图中的不同复杂表达式必须分配不同的别名。例如，查询语句`select b, sum(a + 1) as sum_a from table group by b`和`select b, sum(a) as sum_a from table group by b`不能用于为同一基表创建同步物化视图。你可以为复杂表达式设置不同的别名。
  >   - 你可以通过执行`EXPLAIN <sql_statement>`来检查是否通过复杂表达式创建的同步物化视图重写了查询。有关更多信息，请参见[查询分析](../../../administration/Query_planning.md)。

- WHERE（可选）

  从v3.2开始，同步物化视图支持可以筛选用于物化视图的行的WHERE子句。

- GROUP BY（可选）

  查询的GROUP BY列。如果未指定此参数，则数据将不会默认进行分组。

- ORDER BY（可选）

  查询的ORDER BY列。

  - ORDER BY子句中的列必须按照`select_expr`中列的相同顺序进行声明。
  - 如果查询语句包含GROUP BY子句，则ORDER BY列必须与GROUP BY列相同。
  - 如果未指定此参数，则系统将根据以下规则自动补充ORDER BY列：
    - 如果物化视图为AGGREGATE类型，则所有GROUP BY列会自动用作排序键。
    - 如果物化视图不是AGGREGATE类型，则StarRocks根据前缀列自动选择排序键。

### 查询同步物化视图

由于同步物化视图本质上是基表的索引，而不是物理表，因此只能使用提示`[_SYNC_MV_]`查询同步物化视图：

```SQL
-- 不要忽略提示中的方括号[]
SELECT * FROM <mv_name> [_SYNC_MV_];
```

> **注意**
>
> 目前，即使为同步物化视图指定了别名，StarRocks也会自动生成列名。

### 使用同步物化视图进行自动查询重写

执行遵循同步物化视图模式的查询时，原始查询语句会自动重写，并使用物化视图中存储的中间结果。

下表显示了原始查询中的聚合函数与用于构建物化视图的聚合函数之间的对应关系。根据业务场景，您可以选择相应的聚合函数来构建物化视图。

| **原始查询中的聚合函数**         | **物化视图中的聚合函数** |
| -------------------------------- | ------------------------ |
| sum                              | sum                      |
| min                              | min                      |
| max                              | max                      |
| count                            | count                    |
| bitmap_union、bitmap_union_count、count(distinct) | bitmap_union |
| hll_raw_agg、hll_union_agg、ndv、approx_count_distinct | hll_union |
| percentile_approx、percentile_union | percentile_union |

## 异步物化视图

### 语法

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
-- distribution_desc
[DISTRIBUTED BY HASH(<bucket_key>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]]
-- refresh_desc
[REFRESH 
-- refresh_moment
    [IMMEDIATE | DEFERRED]
-- refresh_scheme
    [ASYNC [START (<start_time>)] [EVERY (INTERVAL <refresh_interval>)] | MANUAL]
]
-- partition_expression
[PARTITION BY 
    {<date_column> | date_trunc(fmt, <date_column>)}
]
-- order_by_expression
[ORDER BY (<sort_key>)]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

方括号中的参数为可选项。

### 参数

**mv_name**（必需）

物化视图的名称。名称要求如下：

- 名称必须由字母（a-z或A-Z）、数字（0-9）或下划线（_）组成，并且只能以字母开头。
- 名称的长度不能超过64个字符。
- 名称区分大小写。

> **注意**
>
> 可以在同一数据库中在同一基础表上创建多个物化视图，但是无法将在同一数据库中的物化视图命名重复。

**注释**（可选）

对物化视图进行评论。请注意，`COMMENT` 必须放置在 `mv_name` 之后。否则，无法创建物化视图。

**distribution_desc**（可选）

异步物化视图的分桶策略。StarRocks支持哈希分桶和随机分桶（自v3.1起）。如果不指定此参数，StarRocks将使用随机分桶策略，并自动设置桶的数量。

> **注意**
>
> 在创建异步物化视图时，必须指定 `distribution_desc` 或 `refresh_scheme`，或两者都指定。

- **哈希分桶**：

  语法

  ```SQL
  DISTRIBUTED BY HASH (<bucket_key1>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]
  ```

  更多信息，请参阅 [数据分布](../../../table_design/Data_distribution.md#data-distribution)。

  > **注意**
  >
  > 从v2.5.7开始，当创建表或添加分区时，StarRocks可以自动设置桶的数量（BUCKETS）。您无需手动设置桶的数量。详细信息，请参阅 [确定桶的数量](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)。

- **随机分桶**：

  如果选择随机分桶策略并允许StarRocks自动设置桶的数量，则无需指定 `distribution_desc`。然而，如果希望手动设置桶的数量，可以参考以下语法：

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <bucket_number>
  ```

  > **注意**
  >
  > 采用随机分桶策略的异步物化视图无法分配给一个共位组。

  更多信息，请参阅 [随机分桶](../../../table_design/Data_distribution.md#random-bucketing-since-v31)。

**refresh_moment**（可选）

物化视图的刷新时刻。默认值为 `IMMEDIATE`。有效值：

- `IMMEDIATE`：在创建后立即刷新异步物化视图。
- `DEFERRED`：异步物化视图在创建后不会被刷新。用户可以手动刷新物化视图或安排定期刷新任务。

**refresh_scheme**（可选）

> **注意**
>
> 在创建异步物化视图时，必须指定 `distribution_desc` 或 `refresh_scheme`，或两者都指定。

异步物化视图的刷新策略。有效值：

- `ASYNC`：异步刷新模式。每次基础表数据更改时，根据预定义的刷新间隔自动刷新物化视图。您还可以进一步指定刷新开始时间为 `START('yyyy-MM-dd hh:mm:ss')`，并且使用以下单位指定刷新间隔为 `EVERY (interval n day/hour/minute/second)`：`DAY`、`HOUR`、`MINUTE` 和 `SECOND`。例如：`ASYNC START ('2023-09-12 16:30:25') EVERY (INTERVAL 5 MINUTE)`。如果不指定间隔，将使用默认值 `10 MINUTE`。
- `MANUAL`：手动刷新模式。物化视图将不会自动刷新。刷新任务只能由用户手动触发。

如果未指定此参数，则默认值为 `MANUAL`。

**partition_expression**（可选）

异步物化视图的分区策略。就目前StarRocks的版本而言，创建异步物化视图时仅支持一个分区表达式。

> **注意**
>
> 目前，异步物化视图不支持列表分区策略。

有效值：

- `column_name`：用于分区的列的名称。表达式 `PARTITION BY dt` 意味着根据 `dt` 列对物化视图进行分区。
- `date_trunc` function：用于截断时间单位的函数。`PARTITION BY date_trunc("MONTH", dt)` 表示`dt`列被截断为月份单位进行分区。`date_trunc` 函数支持以 `YEAR`、`MONTH`、`DAY`、`HOUR` 和 `MINUTE` 为单位进行时间截断。
- `str2date` function：用于将基础表的字符串类型分区进行物化视图的分区。`PARTITION BY str2date(dt, "%Y%m%d")` 表示`dt`列是一个字符串日期类型，其日期格式为 `"%Y%m%d"`。`str2date` 函数支持大量的日期格式，可参考 [str2date](../../sql-functions/date-time-functions/str2date.md) 获取更多信息。自v3.1.4开始支持。
- `time_slice` 或 `date_slice` 函数：自v3.1版本开始，您还可以使用这些函数将给定时间转换为基于指定时间粒度的时间间隔的开头或结尾，例如 `PARTITION BY date_trunc("MONTH", time_slice(dt, INTERVAL 7 DAY))`，其中 `time_slice` 和 `date_slice` 必须具有比 `date_trunc` 更精细的粒度。您可以用它们指定一个比分区键的粒度更细的 GROUP BY 列，例如 `GROUP BY time_slice(dt, INTERVAL 1 MINUTE) PARTITION BY date_trunc('DAY', ts)`。

如果未指定此参数，默认情况下不采用分区策略。

**order_by_expression**（可选）

异步物化视图的排序关键字。如果不指定排序关键字，StarRocks将从SELECT列中选择一些前缀列作为排序关键字。例如，在 `select a, b, c, d` 中，排序关键字可以是 `a` 和 `b`。该参数自StarRocks v3.0起开始支持。

**属性**（可选）

异步物化视图的属性。您可以使用 [ALTER MATERIALIZED VIEW](./ALTER_MATERIALIZED_VIEW.md) 修改已存在的物化视图的属性。

- `session.`：如果要修改物化视图相关属性的会话变量，必须在属性前加上 `session.` 前缀，例如 `session.query_timeout`。对于非会话属性，例如 `mv_rewrite_staleness_second`，无需指定前缀。
- `replication_num`：要创建的物化视图副本的数量。
- `storage_medium`：存储介质类型。有效值：`HDD` 和 `SSD`。
- `storage_cooldown_time`：分区的存储冷却时间。如果同时使用HDD和SSD存储介质，指定此属性后，SSD存储中的数据将在指定的时间后移动到HDD存储。格式：“yyyy-MM-dd HH:mm:ss”。指定的时间必须晚于当前时间。如果未明确指定该属性，则默认情况下不执行存储冷却。
- `partition_ttl`：分区的生存周期（TTL）。在指定的时间范围内仍存在数据的分区将被保留。过期分区将被自动删除。单位：`YEAR`、`MONTH`、`DAY`、`HOUR` 和 `MINUTE`。例如，您可以将此属性指定为 `2 MONTH`。从v3.1.5开始支持该属性。
- `partition_ttl_number`：要保留的最近物化视图分区的数量。对于那些开始时间早于当前时间的分区，在这些分区的数量超过此值后，较不常见的分区将被删除。StarRocks将根据前端配置项`dynamic_partition_check_interval_seconds`中指定的时间间隔定期检查物化视图分区，并自动删除过期分区。如果启用了[动态分区](../../../table_design/dynamic_partitioning.md) 策略，则事先创建的分区不计入在内。当值为`-1`时，将保留物化视图的所有分区。默认值：`-1`。
- `partition_refresh_number`：单次刷新的最大分区数量。如果要刷新的分区数量超过此值，StarRocks将拆分刷新任务并分批完成。仅当上一批分区成功刷新后，StarRocks才会继续刷新下一批分区，直到所有分区被刷新。如果任何分区刷新失败，则不会生成后续刷新任务。当值为`-1`时，刷新任务不会被拆分。默认值：`-1`。
- `excluded_trigger_tables`: 如果在这里列出了物化视图的基表，则当基表中的数据发生变化时，自动刷新任务不会被触发。此参数仅适用于加载触发的刷新策略，并且通常与属性`auto_refresh_partitions_limit`一起使用。格式：`[db_name.]table_name`。当值为空字符串时，所有基表的数据变化都会触发对应物化视图的刷新。默认值为空字符串。
- `auto_refresh_partitions_limit`: 当物化视图刷新被触发时，需要刷新的最近物化视图分区的数量。您可以使用此属性来限制刷新范围并减少刷新成本。但是，由于并非所有分区都被刷新，所以物化视图中的数据可能与基表不一致。默认值：`-1`。当值为`-1`时，所有分区将被刷新。当值为正整数N时，StarRocks按时间顺序对现有分区进行排序，并刷新当前分区和最近的N-1个分区。如果分区数量小于N，则StarRocks刷新所有现有分区。如果您的物化视图中提前创建了动态分区，StarRocks将刷新所有预先创建的分区。
- `mv_rewrite_staleness_second`: 如果物化视图的最后刷新在此属性指定的时间间隔内，则无论基表中的数据是否发生变化，都可以直接使用此物化视图进行查询重写。如果最后刷新在此时间间隔之前，则StarRocks会检查基表是否已更新，以确定是否可以使用物化视图进行查询重写。单位：秒。从v3.0开始支持此属性。
- `colocate_with`：异步物化视图的集群位置组。有关更多信息，请参见[Colocate Join](../../../using_starrocks/Colocate_join.md)。从v3.0开始支持此属性。
- `unique_constraints` 和 `foreign_key_constraints`：在View Delta Join场景中为了查询重写而创建异步物化视图时的唯一键约束和外键约束。有关更多信息，请参见[Asynchronous materialized view - Rewrite queries in View Delta Join scenario](../../../using_starrocks/query_rewrite_with_materialized_views.md)。从v3.0开始支持此属性。
- `resource_group`：物化视图刷新任务所属的资源组。有关资源组的更多信息，请参见[资源组](../../../administration/resource_group.md)。
- `query_rewrite_consistency`：与异步物化视图的查询重写规则。从v3.2开始支持此属性。有效值：
 - `disable`：禁用异步物化视图的自动查询重写。
 - `checked`（默认值）：仅当物化视图满足实时性要求时才启用自动查询重写，也就是说：
  - 如果未指定`mv_rewrite_staleness_second`，则只有当物化视图的数据与所有基表中的数据一致时，才能对其进行查询重写。
  - 如果指定了`mv_rewrite_staleness_second`，则只有当其最后刷新在失效时间间隔内时，才能对其进行查询重写。
 - `loose`：直接启用自动查询重写，无需进行一致性检查。
- `force_external_table_query_rewrite`：是否启用外部目录的基础异步物化视图的查询重写。从v3.2开始支持此属性。有效值：
 - `true`：启用外部目录的基础异步物化视图的查询重写。
 - `false`（默认值）：禁用外部目录的基础异步物化视图的查询重写。

  由于无法保证基表和外部目录的物化视图之间的强数据一致性，默认情况下将此特性设置为`false`。启用此特性后，将根据`query_rewrite_consistency`中指定的规则来进行查询重写。

> **注意**
>
> 唯一键约束和外键约束仅用于查询重写。在将数据加载到表中时，外键约束检查是不能保证的。您必须确保加载到表中的数据满足约束条件。

**query_statement**（必需）

用于创建异步物化视图的查询语句。

> **注意**
>
> 目前，StarRocks不支持使用列表分区策略创建异步物化视图。

### 查询异步物化视图

异步物化视图是一个物理表。您可以像操作常规表一样操作它，**唯一的例外是您不能直接将数据加载到异步物化视图中**。

### 异步物化视图的自动查询重写

StarRocks v2.5支持基于SPJG类型的异步物化视图的自动、透明查询重写。SPJG类型的物化视图指的是其计划仅包括扫描、过滤、投影和聚合类型的运算符的物化视图。SPJG类型的物化视图查询重写包括单表查询重写、连接查询重写、聚合查询重写、联合查询重写以及基于嵌套物化视图的查询重写。

有关更多信息，请参见[Asynchronous materialized view - Rewrite queries with the asynchronous materialized view](../../../using_starrocks/query_rewrite_with_materialized_views.md)。

### 支持的数据类型

- 基于StarRocks默认目录创建的异步物化视图支持以下数据类型：

 - **日期**：DATE，DATETIME
 - **字符串**：CHAR，VARCHAR
 - **数值**：BOOLEAN，TINYINT，SMALLINT，INT，BIGINT，LARGEINT，FLOAT，DOUBLE，DECIMAL，PERCENTILE
 - **半结构化**：ARRAY，JSON，MAP（从v3.1开始支持），STRUCT（从v3.1开始支持）
 - **其他**：BITMAP，HLL

> **注意**
>
> BITMAP，HLL和PERCENTILE自v2.4.5开始支持。

- 基于StarRocks外部目录创建的异步物化视图支持以下数据类型：

 - Hive目录

   - **数值**：INT/INTEGER，BIGINT，DOUBLE，FLOAT，DECIMAL
   - **日期**：TIMESTAMP
   - **字符串**：STRING，VARCHAR，CHAR
   - **半结构化**：ARRAY

 - Hudi目录

   - **数值**：BOOLEAN，INT，LONG，FLOAT，DOUBLE，DECIMAL
   - **日期**：DATE，TimeMillis/TimeMicros，TimestampMillis/TimestampMicros
   - **字符串**：STRING
   - **半结构化**：ARRAY

 - Iceberg目录

   - **数值**：BOOLEAN，INT，LONG，FLOAT，DOUBLE，DECIMAL（P，S）
   - **日期**：DATE，TIME，TIMESTAMP
   - **字符串**：STRING，UUID，FIXED（L），BINARY
   - **半结构化**：LIST

## 使用注意事项

- 当前版本的StarRocks不支持同时创建多个物化视图。只有在前一个物化视图完成后才能创建新的物化视图。

- 关于同步物化视图：

 - 同步物化视图仅支持对单列进行聚合函数。形如`sum(a+b)`的查询语句不受支持。
 - 同步物化视图仅支持基表每列的一个聚合函数。形如`select sum(a), min(a) from table`的查询语句不受支持。
 - 创建同步物化视图时，必须指定GROUP BY子句，并且在SELECT中至少指定一个GROUP BY列。
 - 同步物化视图不支持JOIN等子句和GROUP BY的HAVING子句。
 - 使用ALTER TABLE DROP COLUMN删除基表的某一列时，您必须确保所有基表的同步物化视图不包含被删除的列，否则删除操作将失败。在删除列之前，您必须先删除包含该列的所有同步物化视图。
 - 为一个表创建太多同步物化视图将影响数据加载效率。当数据正在加载到基表时，同步物化视图中的数据和基表的数据将同步更新。如果一个基表包含`n`个同步物化视图，那么数据加载到基表中的效率与加载到`n`个表的效率大致相同。

- 关于嵌套异步物化视图：

 - 每个物化视图的刷新策略仅适用于对应的物化视图。
 - 目前，StarRocks不限制嵌套级别的数量。在生产环境中，建议嵌套层级的数量不要超过三层。

- 关于外部目录异步物化视图：

 - 外部目录物化视图仅支持异步固定间隔刷新和手动刷新。
 - 无法保证外部目录的物化视图与基表之间的严格一致性。
 - 目前不支持基于外部资源构建物化视图。
 - 目前，StarRocks无法感知外部目录中基表数据是否发生变化，因此每次刷新基表时默认会刷新所有分区。您可以使用[REFRESH MATERIALIZED VIEW](../data-manipulation/REFRESH_MATERIALIZED_VIEW.md)手动刷新部分分区。

## 示例

### 同步物化视图示例

基表的架构如下：

```Plain Text
mysql> desc duplicate_table;
+-------+--------+------+------+---------+-------+
```
| 字段       | 类型        | 可空 | 键   | 默认值 | 扩展    |
+-----------+-------------+------+-----+--------+-------+
| k1        | INT         | 是   | true| N/A    |        |
| k2        | INT         | 是   | true| N/A    |        |
| k3        | BIGINT      | 是   | true| N/A    |        |
| k4        | BIGINT      | 是   | true| N/A    |        |
+-----------+-------------+------+-----+--------+-------+
```

Example 1: 创建一个仅包含原始表的列（k1，k2）的同步物化视图。

```sql
create materialized view k1_k2 as
select k1, k2 from duplicate_table;
```

物化视图仅包含两列k1和k2，没有任何聚合。

```plain text
+-----------------+-------+--------+------+-------+--------+-------+
| 索引名         | 字段  | 类型    | 可空 | 键   | 默认值 | 扩展  |
+-----------------+--------+------+-----+--------+---------+-------+
| k1_k2           | k1    | INT    | 是   | true   | N/A     |       |
|                 | k2    | INT    | 是   | true   | N/A     |       |
+-----------------+-------+--------+------+-------+---------+-------+
```

Example 2: 创建一个按k2排序的同步物化视图。

```sql
create materialized view k2_order as
select k2, k1 from duplicate_table order by k2;
```

物化视图的模式如下所示。物化视图仅包含两列k2和k1，其中列k2是一个排序列，没有任何聚合。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| 索引名         | 字段  | 类型    | 可空 | 键  | 默认值 | 扩展  |
+-----------------+-------+--------+------+-------+---------+-------+
| k2_order        | k2    | INT    | 是   | true  | N/A    |       |
|                 | k1    | INT    | 是   | false | N/A    | NONE  |
+-----------------+-------+--------+------+-------+---------+-------+
```

Example 3: 创建一个按k1和k2分组，并对k3进行SUM聚合的同步物化视图。

```sql
create materialized view k1_k2_sumk3 as
select k1, k2, sum(k3) from duplicate_table group by k1, k2;
```

物化视图的模式如下所示。物化视图包含三列k1，k2和sum(k3)，其中k1，k2是分组列，sum(k3)是根据k1和k2分组的k3列的总和。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| 索引名         | 字段  | 类型    | 可空 | 键  | 默认值 | 扩展  |
+-----------------+-------+--------+------+-------+---------+-------+
| k1_k2_sumk3     | k1    | INT    | 是   | true  | N/A     |       |
|                 | k2    | INT    | 是   | true  | N/A     |       |
|                 | k3    | BIGINT | 是   | false | N/A     | SUM   |
+-----------------+-------+--------+------+-------+---------+-------+
```

因为物化视图没有声明排序列，并采用了聚合函数，StarRocks默认补充了分组列k1和k2。

Example 4: 创建一个同步物化视图以删除重复行。

```sql
create materialized view deduplicate as
select k1, k2, k3, k4 from duplicate_table group by k1, k2, k3, k4;
```

物化视图的模式如下所示。物化视图包含k1，k2，k3和k4列，没有重复行。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| 索引名         | 字段  | 类型    | 可空 | 键  | 默认值 | 扩展  |
+-----------------+-------+--------+------+-------+---------+-------+
| deduplicate     | k1    | INT    | 是   | true  | N/A     |       |
|                 | k2    | INT    | 是   | true  | N/A     |       |
|                 | k3    | BIGINT | 是   | true  | N/A     |       |
|                 | k4    | BIGINT | 是   | true  | N/A     |       |
+-----------------+-------+--------+------+-------+---------+-------+
```

Example 5: 创建一个不含排序列的非聚合同步物化视图。

基本表的模式如下所示：

```plain text
+-------+-----------+------+-------+---------+-------+
| 字段 | 类型       | 可空 | 键   | 默认值 | 扩展   |
+-------+-----------+------+-------+---------+-------+
| k1    | TINYINT   | 是   | true  | N/A     |       |
| k2    | SMALLINT  | 是   | true  | N/A     |       |
| k3    | INT       | 是   | true  | N/A     |       |
| k4    | BIGINT    | 是   | true  | N/A     |       |
| k5    | DECIMAL(9,0) | 是 | true | N/A    |       |
| k6    | DOUBLE    | 是   | false | N/A   | NONE   |
| k7    | VARCHAR(20) | 是 | false | N/A   | NONE   |
+-------+-----------+------+-------+---------+-------+
```

物化视图包含k3，k4，k5，k6和k7列，并且未声明排序列。使用以下语句创建物化视图：

```sql
create materialized view mv_1 as
select k3, k4, k5, k6, k7 from all_type_table;
```

StarRocks自动默认使用k3，k4和k5作为排序列。这三列类型占用的字节的总和是4（INT）+8（BIGINT）+16（DECIMAL）= 28 < 36。因此，这三列被添加为排序列。

物化视图的模式如下所示。

```plain text
+----------------+-------+-----------+------+-------+---------+-------+
| 索引名        | 字段 | 类型       | 可空 | 键   | 默认值 | 扩展   |
+----------------+-------+-----------+------+-------+---------+-------+
| mv_1           | k3    | INT       | 是   | true  | N/A     |       |
|                | k4    | BIGINT    | 是   | true  | N/A     |       |
|                | k5    | DECIMAL(9,0) | 是 | true | N/A    |       |
|                | k6    | DOUBLE    | 是   | false | N/A    | NONE  |
|                | k7    | VARCHAR(20) | 是 | false | N/A    | NONE  |
+----------------+-------+-----------+------+-------+---------+-------+
```

可以观察到k3，k4和k5列的`key`字段为`true`，表示它们是排序键。k6和k7列的`key`字段为`false`，表示它们不是排序键。

Example 6: 创建一个包含WHERE子句和复杂表达式的同步物化视图。

```SQL
-- 创建基本表：user_event
CREATE TABLE user_event (
      ds date   NOT NULL,
      id  varchar(256)    NOT NULL,
      user_id int DEFAULT NULL,
      user_id1    varchar(256)    DEFAULT NULL,
      user_id2    varchar(256)    DEFAULT NULL,
      column_01   int DEFAULT NULL,
      column_02   int DEFAULT NULL,
      column_03   int DEFAULT NULL,
      column_04   int DEFAULT NULL,
      column_05   int DEFAULT NULL,
      column_06   DECIMAL(12,2)   DEFAULT NULL,
      column_07   DECIMAL(12,3)   DEFAULT NULL,
      column_08   JSON   DEFAULT NULL,
      column_09   DATETIME    DEFAULT NULL,
      column_10   DATETIME    DEFAULT NULL,
      column_11   DATE    DEFAULT NULL,
      column_12   varchar(256)    DEFAULT NULL,
      column_13   varchar(256)    DEFAULT NULL,
      column_14   varchar(256)    DEFAULT NULL,
      column_15   varchar(256)    DEFAULT NULL,
      column_16   varchar(256)    DEFAULT NULL,
      column_17   varchar(256)    DEFAULT NULL,
      column_18   varchar(256)    DEFAULT NULL,
      column_19   varchar(256)    DEFAULT NULL,
      column_20   varchar(256)    DEFAULT NULL,
      column_21   varchar(256)    DEFAULT NULL,
      column_22   varchar(256)    DEFAULT NULL,
      column_23   varchar(256)    DEFAULT NULL,
      column_24   varchar(256)    DEFAULT NULL,
      column_25   varchar(256)    DEFAULT NULL,
      column_26   varchar(256)    DEFAULT NULL,
      column_27   varchar(256)    DEFAULT NULL,
      column_28   varchar(256)    DEFAULT NULL,
      column_29   varchar(256)    DEFAULT NULL,
      column_30   varchar(256)    DEFAULT NULL,
      column_31   varchar(256)    DEFAULT NULL,
      column_32   varchar(256)    DEFAULT NULL,
      column_33   varchar(256)    DEFAULT NULL,
      column_34   varchar(256)    DEFAULT NULL,
      column_35   varchar(256)    DEFAULT NULL,
      column_36   varchar(256)    DEFAULT NULL,
      column_37   varchar(256)    DEFAULT NULL
  )
  PARTITION BY date_trunc("day", ds)
  DISTRIBUTED BY hash(id);

  -- 使用WHERE子句和复杂表达式创建物化视图。
  创建MATERIALIZED VIEW test_mv1
  AS 
  SELECT
  ds,
  column_19,
  column_36,
  sum(column_01) as column_01_sum,
  bitmap_union(to_bitmap( user_id)) as user_id_dist_cnt,
  bitmap_union(to_bitmap(case when column_01 > 1 and column_34 IN ('1','34')   then user_id2 else null end)) as filter_dist_cnt_1,
  bitmap_union(to_bitmap( case when column_02 > 60 and column_35 IN ('11','13') then  user_id2 else null end)) as filter_dist_cnt_2,
  bitmap_union(to_bitmap(case when column_03 > 70 and column_36 IN ('21','23') then  user_id2 else null end)) as filter_dist_cnt_3,
  bitmap_union(to_bitmap(case when column_04 > 20 and column_27 IN ('31','27') then  user_id2 else null end)) as filter_dist_cnt_4,
  bitmap_union(to_bitmap( case when column_05 > 90 and column_28 IN ('41','43') then  user_id2 else null end)) as filter_dist_cnt_5
  FROM user_event
  WHERE ds >= '2023-11-02'
  GROUP BY
  ds,
  column_19,
  column_36;
 ```

异步物化视图示例

以下示例基于以下基表：

```SQL
创建TABLE线路订单（
  `lo_orderkey` int(11) NOT NULL COMMENT "",
  `lo_linenumber` int(11) NOT NULL COMMENT "",
  `lo_custkey` int(11) NOT NULL COMMENT "",
  `lo_partkey` int(11) NOT NULL COMMENT "",
  `lo_suppkey` int(11) NOT NULL COMMENT "",
  `lo_orderdate` int(11) NOT NULL COMMENT "",
  `lo_orderpriority` varchar(16) NOT NULL COMMENT "",
  `lo_shippriority` int(11) NOT NULL COMMENT "",
  `lo_quantity` int(11) NOT NULL COMMENT "",
  `lo_extendedprice` int(11) NOT NULL COMMENT "",
  `lo_ordtotalprice` int(11) NOT NULL COMMENT "",
  `lo_discount` int(11) NOT NULL COMMENT "",
  `lo_revenue` int(11) NOT NULL COMMENT "",
  `lo_supplycost` int(11) NOT NULL COMMENT "",
  `lo_tax` int(11) NOT NULL COMMENT "",
  `lo_commitdate` int(11) NOT NULL COMMENT "",
  `lo_shipmode` varchar(11) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`lo_orderkey`)
评论“OLAP”
PARTITION BY RANGE(`lo_orderdate`)
（PARTITION p1 VALUES [("-2147483648"), ("19930101")),
PARTITION p2 VALUES [("19930101"), ("19940101")),
PARTITION p3 VALUES [("19940101"), ("19950101")),
PARTITION p4 VALUES [("19950101"), ("19960101")),
PARTITION p5 VALUES [("19960101"), ("19970101")),
PARTITION p6 VALUES [("19970101"), ("19980101")),
PARTITION p7 VALUES [("19980101"), ("19990101")))
DISTRIBUTED BY HASH(`lo_orderkey`);

CREATE TABLE IF NOT EXISTS客户`（
  `c_custkey`` int(11) NOT NULL COMMENT "",
  `c_name` varchar(26) NOT NULL COMMENT "",
  `c_address` varchar(41) NOT NULL COMMENT "",
  `c_city` varchar(11) NOT NULL COMMENT "",
  `c_nation` varchar(16) NOT NULL COMMENT "",
  `c_region` varchar(13) NOT NULL COMMENT "",
  `c_phone` varchar(16) NOT NULL COMMENT "",
  `c_mktsegment` varchar(11) NOT NULL COMMENT ""
）ENGINE=OLAP
DUPLICATE KEY(`c_custkey`)
COMMENT“OLAP”
DISTRIBUTED BY HASH(`c_custkey`);

CREATE TABLE IF NOT EXISTS `dates` (
  `d_datekey` int(11) NOT NULL COMMENT "",
  `d_date` varchar(20) NOT NULL COMMENT "",
  `d_dayofweek` varchar(10) NOT NULL COMMENT "",
  `d_month` varchar(11) NOT NULL COMMENT "",
  `d_year` int(11) NOT NULL COMMENT "",
  `d_yearmonthnum` int(11) NOT NULL COMMENT "",
  `d_yearmonth` varchar(9) NOT NULL COMMENT "",
  `d_daynuminweek` int(11) NOT NULL COMMENT "",
  `d_daynuminmonth` int(11) NOT NULL COMMENT "",
  `d_daynuminyear` int(11) NOT NULL COMMENT "",
  `d_monthnuminyear` int(11) NOT NULL COMMENT "",
  `d_weeknuminyear` int(11) NOT NULL COMMENT "",
  `d_sellingseason` varchar(14) NOT NULL COMMENT "",
  `d_lastdayinweekfl` int(11) NOT NULL COMMENT "",
  `d_lastdayinmonthfl` int(11) NOT NULL COMMENT "",
  `d_holidayfl` int(11) NOT NULL COMMENT "",
  `d_weekdayfl` int(11) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`d_datekey`)
COMMENT“OLAP”
DISTRIBUTED BY HASH(`d_datekey`);

CREATE TABLE IF NOT EXISTS `supplier` (
  `s_suppkey` int(11) NOT NULL COMMENT "",
  `s_name` varchar(26) NOT NULL COMMENT "",
  `s_address` varchar(26) NOT NULL COMMENT "",
  `s_city` varchar(11) NOT NULL COMMENT "",
  `s_nation` varchar(16) NOT NULL COMMENT "",
  `s_region` varchar(13) NOT NULL COMMENT "",
  `s_phone` varchar(16) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`s_suppkey`)
COMMENT“OLAP”
DISTRIBUTED BY HASH(`s_suppkey`);

CREATE TABLE IF NOT EXISTS `part` (
  `p_partkey` int(11) NOT NULL COMMENT "",
  `p_name` varchar(23) NOT NULL COMMENT "",
  `p_mfgr` varchar(7) NOT NULL COMMENT "",
  `p_category` varchar(8) NOT NULL COMMENT "",
  `p_brand` varchar(10) NOT NULL COMMENT "",
  `p_color` varchar(12) NOT NULL COMMENT "",
  `p_type` varchar(26) NOT NULL COMMENT "",
  `p_size` int(11) NOT NULL COMMENT "",
  `p_container` varchar(11) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`p_partkey`)
COMMENT“OLAP”
DISTRIBUTED BY HASH(`p_partkey`);

创建订单表（
    dt date NOT NULL,
    order_id bigint NOT NULL,
    user_id int NOT NULL,
    merchant_id int NOT NULL,
    good_id int NOT NULL,
    good_name string NOT NULL,
    price int NOT NULL,
    cnt int NOT NULL,
    revenue int NOT NULL,
    state tinyint NOT NULL
）
PRIMARY KEY（dt，order_id）
PARTITION BY RANGE（`dt`）
（ PARTITION p20210820 VALUES [('2021-08-20'), ('2021-08-21')),
PARTITION p20210821 VALUES [('2021-08-21'), ('2021-08-22')) ）
DISTRIBUTED BY HASH（order_id）
属性（
    "replication_num" = "3",
    "enable_persistent_index" = "true"
);
```

示例1：创建非分区物化视图。

```SQL
CREATE MATERIALIZED VIEW lo_mv1
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH ASYNC
AS
select
    lo_orderkey, 
    lo_custkey, 
    sum(lo_quantity) as total_quantity, 
    sum(lo_revenue) as total_revenue, 
    count(lo_shipmode) as shipmode_count
from lineorder 
group by lo_orderkey, lo_custkey 
order by lo_orderkey;
```

示例2：创建分区物化视图。

```SQL
CREATE MATERIALIZED VIEW lo_mv2
PARTITION BY `lo_orderdate`
DISTRIBUTED BY HASH(`lo_orderkey`)
```SQL
REFRESH ASYNC START('2023-07-01 10:00:00') EVERY (interval 1 day)
AS
select
    lo_orderkey,
    lo_orderdate,
    lo_custkey, 
    sum(lo_quantity) as total_quantity, 
    sum(lo_revenue) as total_revenue, 
    count(lo_shipmode) as shipmode_count
from lineorder 
group by lo_orderkey, lo_orderdate, lo_custkey

order by lo_orderkey;

-- Use the date_trunc() function to partition the materialized view by month.
CREATE MATERIALIZED VIEW order_mv1
PARTITION BY date_trunc('month', `dt`)
DISTRIBUTED BY HASH(`order_id`)
REFRESH ASYNC START('2023-07-01 10:00:00') EVERY (interval 1 day)
AS
select
    dt,
    order_id,
    user_id,
    sum(cnt) as total_cnt,
    sum(revenue) as total_revenue, 
    count(state) as state_count
from orders

group by dt, order_id, user_id;

```

```SQL
CREATE MATERIALIZED VIEW flat_lineorder
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH MANUAL
AS
SELECT
    l.LO_ORDERKEY AS LO_ORDERKEY,
    l.LO_LINENUMBER AS LO_LINENUMBER,
    l.LO_CUSTKEY AS LO_CUSTKEY,
    l.LO_PARTKEY AS LO_PARTKEY,
    l.LO_SUPPKEY AS LO_SUPPKEY,
    l.LO_ORDERDATE AS LO_ORDERDATE,
    l.LO_ORDERPRIORITY AS LO_ORDERPRIORITY,
    l.LO_SHIPPRIORITY AS LO_SHIPPRIORITY,
    l.LO_QUANTITY AS LO_QUANTITY,
    l.LO_EXTENDEDPRICE AS LO_EXTENDEDPRICE,
    l.LO_ORDTOTALPRICE AS LO_ORDTOTALPRICE,
    l.LO_DISCOUNT AS LO_DISCOUNT,
    l.LO_REVENUE AS LO_REVENUE,
    l.LO_SUPPLYCOST AS LO_SUPPLYCOST,
    l.LO_TAX AS LO_TAX,
    l.LO_COMMITDATE AS LO_COMMITDATE,
    l.LO_SHIPMODE AS LO_SHIPMODE,
    c.C_NAME AS C_NAME,
    c.C_ADDRESS AS C_ADDRESS,
    c.C_CITY AS C_CITY,
    c.C_NATION AS C_NATION,
    c.C_REGION AS C_REGION,
    c.C_PHONE AS C_PHONE,
    c.C_MKTSEGMENT AS C_MKTSEGMENT,
    s.S_NAME AS S_NAME,
    s.S_ADDRESS AS S_ADDRESS,
    s.S_CITY AS S_CITY,
    s.S_NATION AS S_NATION,
    s.S_REGION AS S_REGION,
    s.S_PHONE AS S_PHONE,
    p.P_NAME AS P_NAME,
    p.P_MFGR AS P_MFGR,
    p.P_CATEGORY AS P_CATEGORY,
    p.P_BRAND AS P_BRAND,
    p.P_COLOR AS P_COLOR,
    p.P_TYPE AS P_TYPE,
    p.P_SIZE AS P_SIZE,
    p.P_CONTAINER AS P_CONTAINER FROM lineorder AS l 
INNER JOIN customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
INNER JOIN supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY

INNER JOIN part AS p ON p.P_PARTKEY = l.LO_PARTKEY;

```

```SQL
-- Create a partitioned materialized view and use `str2date` to transform the STRING type partition key of the base table into the date type as for the materialized view.
-- Hive Table with string partition column.
CREATE TABLE `part_dates` (
  `d_date` varchar(20) DEFAULT NULL,
  `d_dayofweek` varchar(10) DEFAULT NULL,
  `d_month` varchar(11) DEFAULT NULL,
  `d_year` int(11) DEFAULT NULL,
  `d_yearmonthnum` int(11) DEFAULT NULL,
  `d_yearmonth` varchar(9) DEFAULT NULL,
  `d_daynuminweek` int(11) DEFAULT NULL,
  `d_daynuminmonth` int(11) DEFAULT NULL,
  `d_daynuminyear` int(11) DEFAULT NULL,
  `d_monthnuminyear` int(11) DEFAULT NULL,
  `d_weeknuminyear` int(11) DEFAULT NULL,
  `d_sellingseason` varchar(14) DEFAULT NULL,
  `d_lastdayinweekfl` int(11) DEFAULT NULL,
  `d_lastdayinmonthfl` int(11) DEFAULT NULL,
  `d_holidayfl` int(11) DEFAULT NULL,
  `d_weekdayfl` int(11) DEFAULT NULL,


  `d_datekey` varchar(11) DEFAULT NULL
) partition by (d_datekey);

-- Create the materialied view  with `str2date`.
CREATE MATERIALIZED VIEW IF NOT EXISTS `test_mv` 
PARTITION BY str2date(`d_datekey`,'%Y%m%d')
DISTRIBUTED BY HASH(`d_date`, `d_month`, `d_month`) 
REFRESH MANUAL 
AS
SELECT
`d_date` ,
  `d_dayofweek`,
  `d_month` ,
  `d_yearmonthnum` ,
  `d_yearmonth` ,
  `d_daynuminweek`,
  `d_daynuminmonth`,
  `d_daynuminyear` ,
  `d_monthnuminyear` ,
  `d_weeknuminyear` ,
  `d_sellingseason`,
  `d_lastdayinweekfl`,
  `d_lastdayinmonthfl`,
  `d_holidayfl` ,
  `d_weekdayfl`,
   `d_datekey`