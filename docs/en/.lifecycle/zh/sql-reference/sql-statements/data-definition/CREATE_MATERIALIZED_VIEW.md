---
displayed_sidebar: English
---

# 创建物化视图

## 描述

创建物化视图。有关物化视图的使用信息，请参阅[同步物化视图](../../../using_starrocks/Materialized_view-single_table.md)和[异步物化视图](../../../using_starrocks/Materialized_view.md)。

> **注意**
>
> 只有在基表所在的数据库中具有 CREATE MATERIALIZED VIEW 权限的用户才能创建物化视图。

创建物化视图是一个异步操作。成功运行此命令表示物化视图创建任务已成功提交。您可以通过[SHOW ALTER MATERIALIZED VIEW](../data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)命令在数据库中查看同步物化视图的构建状态，并通过查询元数据视图[`tasks`](../../../reference/information_schema/tasks.md)和[`task_runs`](../../../reference/information_schema/task_runs.md)来查询异步物化视图的构建状态。

StarRocks 从 v2.4 开始支持异步物化视图。在以前的版本中，异步物化视图和同步物化视图之间的主要区别如下：

|                       | **单表聚合** | **多表联接** | **查询重写** | **刷新策略** | **基表** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **异步 MV** | 是 | 是 | 是 | <ul><li>异步刷新 </li><li>手动刷新</li></ul> | 来自以下多个表：<ul><li>默认</li><li>目录 外部目录 （v2.5）现有物化视图 （v2.5）现有视图</li><li> （v3.1）</li><li> </li></ul>|
| **同步 MV（汇总）**  | 聚合函数的选择有限 | 不 | 是 | 数据加载期间的同步刷新 | 默认目录中的单个表 |

## 同步物化视图

### 语法

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

方括号中的参数 [] 是可选的。

### 参数

**mv_name** （必填）

物化视图的名称。命名要求如下：

- 名称必须由字母（a-z 或 A-Z）、数字 （0-9） 或下划线 （\_） 组成，并且只能以字母开头。
- 名称的长度不能超过 64 个字符。
- 名称区分大小写。

**COMMENT** （可选）

对物化视图进行注释。请注意， `COMMENT` 必须放在 `mv_name`之后。否则，无法创建物化视图。

**query_statement** （必填）

用于创建物化视图的查询语句。其结果是物化视图中的数据。语法如下：

```SQL
SELECT select_expr[, select_expr ...]
[WHERE where_expr]
[GROUP BY column_name[, column_name ...]]
[ORDER BY column_name[, column_name ...]]
```

- select_expr （必填）

  查询语句中的所有列，即物化视图架构中的所有列。此参数支持以下值：

  - 简单列或聚合列（如  、 `SELECT a, abs(b), min(c) FROM table_a`其中 、 `a` `b`和 `c` 是基表中列的名称）。如果物化视图未指定列名，StarRocks 会自动为列名指定列名。
  - 表达式（如  、 `SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a`其中 `a+1`、 `b+2` 和 是引用基表中的列的表达式），和 是 `c*c` `x` `y` 分配给物化视图中列的别名。`z` 

  > **注意**
  >
  > - 必须在 中至少指定一列 `select_expr`。
  > - 使用聚合函数创建同步物化视图时，必须指定 GROUP BY 子句，并在 中指定至少一个 GROUP BY 列 `select_expr`。
  > - 同步物化视图不支持 JOIN 和 GROUP BY 的 HAVING 子句等子句。
  > - 从 v3.1 开始，每个同步物化视图可以支持基表每列的多个聚合函数，例如查询语句 `select b, sum(a), min(a) from table group by b`。
  > - 从 v3.1 开始，同步物化视图支持 SELECT 和聚合函数的复杂表达式，例如 `select b, sum(a + 1) as sum_a1, min(cast (a as bigint)) as min_a from table group by b` or `select abs(b) as col1, a + 1 as col2, cast(a as bigint) as col3 from table` 等查询语句。对用于同步物化视图的复杂表达式施加了以下限制：
  >   - 每个复杂表达式都必须具有一个别名，并且必须将不同的别名分配给基表的所有同步物化视图中的不同复杂表达式。例如，查询语句 `select b, sum(a + 1) as sum_a from table group by b` 和 `select b, sum(a) as sum_a from table group by b` 不能用于为同一基表创建同步物化视图。您可以为复杂表达式设置不同的别名。
  >   - 您可以通过执行 来检查您的查询是否被使用复杂表达式创建的同步物化视图重写 `EXPLAIN <sql_statement>`。有关详细信息，请参阅 [查询分析](../../../administration/Query_planning.md)。

- WHERE（可选）

  从 v3.2 开始，同步物化视图支持 WHERE 子句，该子句可以过滤用于物化视图的行。

- GROUP BY（可选）

  查询的 GROUP BY 列。如果未指定该参数，则默认不会对数据进行分组。

- ORDER BY（可选）

  查询的 ORDER BY 列。

  - ORDER BY 子句中的列必须按照与 中的列相同的顺序声明 `select_expr`。
  - 如果查询语句包含 GROUP BY 子句，则 ORDER BY 列必须与 GROUP BY 列相同。
  - 如果不指定该参数，系统将按照以下规则自动补充 ORDER BY 列：
    - 如果物化视图是 AGGREGATE 类型，则所有 GROUP BY 列都将自动用作排序键。
    - 如果物化视图不是 AGGREGATE 类型，StarRocks 会根据前缀列自动选择排序键。

### 查询同步物化视图

由于同步物化视图本质上是基表的索引而不是物理表，因此只能使用提示查询同步物化视图`[_SYNC_MV_]`：

```SQL
-- 不要忽略提示中的方括号 []
SELECT * FROM <mv_name> [_SYNC_MV_];
```

> **谨慎**
>
> 目前，StarRocks 会自动为同步物化视图中的列生成名称，即使您为列指定了别名。

### 使用同步物化视图自动重写查询

执行遵循同步物化视图模式的查询时，将自动重写原始查询语句，并使用物化视图中存储的中间结果。 

下表显示了原始查询中的聚合函数与用于构造物化视图的聚合函数之间的对应关系。您可以根据业务场景选择对应的聚合函数来构建物化视图。

| **原始查询中的聚合函数**           | **物化视图**的聚合函数|
| ------------------------------------------------------ | ----------------------------------------------- |
| 和                                                    | 和                                             |
| 分钟                                                    | 分钟                                             |
| 麦克斯                                                    | 麦克斯                                             |
| 计数                                                  | 计数                                           |
| bitmap_union、bitmap_union_count、计数（非重复）      | bitmap_union                                    |
| hll_raw_agg、hll_union_agg、NDV approx_count_distinct | hll_union                                       |
| percentile_approx， percentile_union                    | percentile_union                                |

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

方括号中的参数 [] 是可选的。

### 参数

**mv_name** （必填）

物化视图的名称。命名要求如下：

- 名称必须由字母（a-z 或 A-Z）、数字 （0-9） 或下划线 （\_） 组成，并且只能以字母开头。
- 名称的长度不能超过 64 个字符。
- 名称区分大小写。

> **谨慎**
>
> 可以在同一个基表上创建多个物化视图，但同一数据库中的物化视图的名称不能重复。

**COMMENT** （可选）

对物化视图进行注释。请注意， `COMMENT` 必须放在 `mv_name`之后。否则，无法创建物化视图。

**distribution_desc** （可选）

异步物化视图的分桶策略。StarRocks 支持哈希分桶和随机分桶（从 v3.1 开始）。如果不指定该参数，StarRocks 会使用随机分桶策略，并自动设置桶数。

> **注意**
>
> 创建异步物化视图时，必须指定 `distribution_desc` 或 `refresh_scheme`，或同时指定两者。

- **哈希分桶**：

  语法

  ```SQL
  DISTRIBUTED BY HASH (<bucket_key1>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]
  ```

  有关详细信息，请参阅[数据分发](../../../table_design/Data_distribution.md#data-distribution)。

  > **注意**
  >
  > 从 v2.5.7 开始，StarRocks 可以在创建表或添加分区时自动设置 BUCKET 数量。您不再需要手动设置存储桶数量。有关详细信息，请参阅[确定存储桶数量](../../../table_design/Data_distribution.md#set-the-number-of-buckets)。

- **随机分桶**：

  如果您选择随机分桶策略，并允许 StarRocks 自动设置分桶数量，则无需指定 `distribution_desc`.但是，如果要手动设置存储桶数量，可以参考以下语法：

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <bucket_number>
  ```

  > **谨慎**
  >
  > 具有随机分桶策略的异步物化视图不能分配给共置组。

  有关更多信息，请参阅[随机分桶](../../../table_design/Data_distribution.md#random-bucketing-since-v31)

**refresh_moment** （可选）

物化视图的刷新时刻。默认值： `IMMEDIATE`。有效值：

- `IMMEDIATE`：创建异步物化视图后立即刷新该视图。
- `DEFERRED`：异步物化视图在创建后不刷新。您可以手动刷新物化视图或安排定期刷新任务。

**refresh_scheme** （可选）

> **注意**
>
> 在创建异步物化视图时，必须指定 `distribution_desc` 或 `refresh_scheme`，或者两者都指定。

异步物化视图的刷新策略。有效值：

- `ASYNC`：异步刷新模式。每当基表数据发生变化时，根据预定义的刷新间隔，物化视图会自动刷新。您还可以进一步指定刷新开始时间为 `START('yyyy-MM-dd hh:mm:ss')`，并使用以下单位指定刷新间隔 `EVERY (interval n day/hour/minute/second)` ：`DAY`、 `HOUR` `MINUTE`、 和 `SECOND`。例如：`ASYNC START ('2023-09-12 16:30:25') EVERY (INTERVAL 5 MINUTE)`。如果未指定间隔，则默认值为 `10 MINUTE`。
- `MANUAL`：手动刷新模式。物化视图不会自动刷新。刷新任务只能由用户手动触发。

如果未指定此参数，则默认值为 `MANUAL`。

**partition_expression** （可选）

异步物化视图的分区策略。就当前 StarRocks 版本而言，在创建异步物化视图时，只支持一个分区表达式。

> **注意**
>
> 目前，异步物化视图不支持列表分区策略。

有效值：

- `column_name`：用于分区的列的名称。表达式 `PARTITION BY dt` 表示根据 `dt` 列对物化视图进行分区。
- `date_trunc` 函数：用于截断时间单位的函数。 `PARTITION BY date_trunc("MONTH", dt)` 表示将 `dt` 列截断为月份作为分区单位。`date_trunc` 函数支持将时间截断为单位 `YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`。
- `str2date` 函数：用于将基表的字符串类型分区划分到物化视图的分区中的函数。 `PARTITION BY str2date(dt, "%Y%m%d")` 表示 `dt` 列是字符串日期类型，其日期格式为 `"%Y%m%d"`。`str2date` 函数支持多种日期格式，更多信息可以参考 [str2date](../../sql-functions/date-time-functions/str2date.md) 。从 v3.1.4 开始支持。
- `time_slice` 或 `date_slice` 函数：从 v3.1 开始，您可以进一步使用这些函数根据指定的时间粒度将给定时间转换为时间间隔的开始或结束，例如， `PARTITION BY date_trunc("MONTH", time_slice(dt, INTERVAL 7 DAY))` 其中 time_slice 和 date_slice 必须具有比 date_trunc 更细的粒度。您可以使用它们来指定粒度比分区键更精细的 GROUP BY 列，例如 `GROUP BY time_slice(dt, INTERVAL 1 MINUTE) PARTITION BY date_trunc('DAY', ts)`。

如果未指定该参数，则默认不采用分区策略。

**order_by_expression** （可选）

异步物化视图的排序键。如果不指定排序键，StarRocks 会从 SELECT 列中选择部分前缀列作为排序键。例如，在 `select a, b, c, d` 中，排序键可以是 `a` 和 `b`。该参数从 StarRocks v3.0 版本开始支持。

**PROPERTIES** （可选）

异步物化视图的属性。您可以使用 [ALTER MATERIALIZED VIEW 修改现有物化视图的属性](./ALTER_MATERIALIZED_VIEW.md)。

- `session.`：如果要修改物化视图的会话变量相关属性，则必须为属性添加 `session.` 前缀，例如 `session.query_timeout`。对于非会话属性，例如 `mv_rewrite_staleness_second`，则无需指定前缀。
- `replication_num`：要创建的物化视图副本数。
- `storage_medium`：存储介质类型。有效值： `HDD` 和 `SSD`。
- `storage_cooldown_time`：分区的存储冷却时间。如果同时使用 HDD 和 SSD 存储介质，则 SSD 存储中的数据将在此属性指定的时间后移动到 HDD 存储。格式：“yyyy-MM-dd HH：mm：ss”。指定的时间必须晚于当前时间。如果未显式指定此属性，则默认情况下不执行存储冷却。
- `partition_ttl`：分区的生存时间 （TTL）。数据在指定时间范围内的分区将被保留。过期的分区会自动删除。单位：`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`。例如，可以将此属性指定为 `2 MONTH`。建议使用此属性。从 v3.1.5 开始支持。
- `partition_ttl_number`：要保留的最新物化视图分区数。对于开始时间早于当前时间的分区，当这些分区的数量超过此值时，将删除较新的分区。StarRocks 会根据 FE 配置项中指定的时间间隔，定期检查物化视图分区`dynamic_partition_check_interval_seconds`，并自动删除过期的分区。如果启用[了动态](../../../table_design/dynamic_partitioning.md)分区策略，则预先创建的分区不计算在内。当值为 `-1` 时，将保留物化视图的所有分区。默认值： `-1`.
- `partition_refresh_number`：在单次刷新中，要刷新的最大分区数。如果需要刷新的分区数量超过此值，StarRocks 会拆分刷新任务并批量完成。只有当上一批分区刷新成功后，StarRocks 才会继续刷新下一批分区，直到所有分区都刷新完毕。如果任何分区刷新失败，则不会生成后续刷新任务。当该值为 `-1` 时，不会拆分刷新任务。默认值： `-1`.
- `excluded_trigger_tables`：如果此处列出了物化视图的基表，则当基表中的数据发生更改时，不会触发自动刷新任务。此参数仅适用于负载触发的刷新策略，通常与属性 `auto_refresh_partitions_limit`一起使用。格式： `[db_name.]table_name`。当该值为空字符串时，所有基表中的任何数据更改都会触发相应物化视图的刷新。默认值为空字符串。
- `auto_refresh_partitions_limit`：触发物化视图刷新时需要刷新的最新物化视图分区数。可以使用此属性来限制刷新范围并降低刷新成本。但是，由于并非所有分区都刷新，因此物化视图中的数据可能与基表不一致。默认值： `-1`。当值为 `-1` 时，将刷新所有分区。当该值为正整数 N 时，StarRocks 会按时间顺序对现有分区进行排序，并刷新当前分区和 N-1 个最近的分区。如果分区数小于 N，StarRocks 会刷新所有已有的分区。如果物化视图中有预先创建的动态分区，StarRocks 会刷新所有预先创建的分区。
- `mv_rewrite_staleness_second`：如果物化视图的上次刷新在此属性中指定的时间间隔内，则无论基表中的数据是否更改，都可以直接使用此物化视图进行查询重写。如果上次刷新时间早于此时间间隔，StarRocks 会检查基表是否已更新，以确定物化视图是否可以用于查询重写。单位：秒。从 v3.0 开始支持此属性。
- `colocate_with`：异步物化视图的共置组。有关详细信息，请参阅 [共置联接](../../../using_starrocks/Colocate_join.md) 。从 v3.0 开始支持此属性。
- `unique_constraints` 和 `foreign_key_constraints`：在视图增量联接方案中为查询重写创建异步物化视图时的唯一键约束和外键约束。有关详细信息，请参阅[异步物化视图 - 重写视图增量联接方案中的查询](../../../using_starrocks/query_rewrite_with_materialized_views.md)。从 v3.0 开始支持此属性。
- `resource_group`：物化视图刷新任务所属的资源组。有关资源组的详细信息，请参阅 [资源组](../../../administration/resource_group.md)。
- `query_rewrite_consistency`：异步物化视图的查询重写规则。从 v3.2 开始支持此属性。有效值：
  - `disable`：禁用异步物化视图的自动查询重写。
  - `checked` （默认值）：仅当物化视图满足时效性要求时，才开启自动查询重写，即：
    - 如果 `mv_rewrite_staleness_second` 未指定，则仅当物化视图的数据与所有基表中的数据一致时，才能使用物化视图进行查询重写。
    - 如果 `mv_rewrite_staleness_second` 指定，则当物化视图的上次刷新在过期时间间隔内时，可以使用物化视图进行查询重写。
  - `loose`：直接启用自动查询重写，无需一致性检查。
- `force_external_table_query_rewrite`：是否为基于目录的外部物化视图启用查询重写。从 v3.2 开始支持此属性。有效值：
  - `true`：为基于目录的外部物化视图启用查询重写。
  - `false` （默认值）：禁用基于外部目录的物化视图的查询重写。

  由于无法保证基表和基于目录的外部物化视图之间的数据一致性，因此默认情况下将此功能设置为 `false`。启用此功能后，将根据 `query_rewrite_consistency` 中指定的规则使用物化视图进行查询重写。

> **注意**
>
> 唯一键约束和外键约束仅用于查询重写。将数据加载到表中时，不能保证外键约束检查。您必须确保加载到表中的数据满足约束条件。

**query_statement** （必填）

用于创建异步物化视图的查询语句。从 v3.1.6 开始，StarRocks 支持使用 CTE 创建异步物化视图。

> **注意**
>
> 目前，StarRocks 不支持使用列表分区策略创建的基表创建异步物化视图。

### 查询异步物化视图

异步物化视图是一个物理表。您可以像操作任何常规表一样操作它 **，但不能直接将数据加载到异步物化视图中**。

### 异步物化视图的自动查询重写

StarRocks v2.5 支持基于 SPJG 类型的异步物化视图进行自动透明的查询重写。SPJG 类型的物化视图是指其计划仅包含 Scan、Filter、Project 和 Aggregate 类型的算子的物化视图。SPJG 类型的物化视图查询重写包括单表查询重写、Join 查询重写、聚合查询重写、联合查询重写和基于嵌套物化视图的查询重写。

有关详细信息，请参阅[异步物化视图 - 使用异步物化视图重写查询](../../../using_starrocks/query_rewrite_with_materialized_views.md)。

### 支持的数据类型

- 基于 StarRocks 默认目录创建的异步物化视图支持以下数据类型：

  - **日期**：DATE、DATETIME
  - **字符串**：CHAR、VARCHAR

  - **数字**：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL、PERCENTILE
  - **半结构化**：ARRAY、JSON、MAP（从 v3.1 开始）、STRUCT（从 v3.1 开始）
  - **其他**：BITMAP、HLL

> **注意**
>
> 自 v2.4.5 起支持 BITMAP、HLL 和 PERCENTILE。

- 基于 StarRocks 外部目录创建的异步物化视图支持以下数据类型：

  - Hive 目录

    - **数字**：INT/INTEGER、BIGINT、DOUBLE、FLOAT、DECIMAL
    - **日期**：时间戳
    - **字符串**：STRING、VARCHAR、CHAR
    - **半结构化**：ARRAY

  - Hudi 目录

    - **数字**：BOOLEAN、INT、LONG、FLOAT、DOUBLE、DECIMAL
    - **日期**：DATE、TimeMillis/TimeMicros、TimestampMillis/TimestampMicros
    - **字符串**：STRING
    - **半结构化**：ARRAY

  - Iceberg 目录

    - **数字**：BOOLEAN、INT、LONG、FLOAT、DOUBLE、DECIMAL(P, S)
    - **日期**：DATE、TIME、TIMESTAMP
    - **字符串**：STRING、UUID、FIXED(L)、BINARY
    - **半结构化**：LIST

## 使用说明

- 当前版本的 StarRocks 不支持同时创建多个物化视图。只有在完成之前的物化视图时，才能创建新的物化视图。

- 关于同步物化视图：

  - 同步物化视图仅支持单个列上的聚合函数。不支持形式为 `sum(a+b)` 的查询语句。
  - 同步物化视图仅支持基表的每一列使用一个聚合函数。不支持查询语句，例如 `select sum(a), min(a) from table`。
  - 创建同步物化视图时，必须指定 GROUP BY 子句，并在 SELECT 中至少指定一个 GROUP BY 列。
  - 同步物化视图不支持 JOIN 和 GROUP BY 的 HAVING 子句等子句。
  - 使用 ALTER TABLE DROP COLUMN 删除基表中的特定列时，必须确保基表的所有同步物化视图都不包含删除的列，否则删除操作将失败。在删除该列之前，必须首先删除包含该列的所有同步物化视图。
  - 为表创建过多的同步物化视图会影响数据加载效率。当数据加载到基表时，同步物化视图和基表中的数据将同步更新。如果基表包含`n`同步物化视图，则将数据加载到基表中的效率与将数据加载到表中的效率大致相同 `n`。

- 关于嵌套异步物化视图：

  - 每个物化视图的刷新策略仅适用于相应的物化视图。
  - 目前，StarRocks 对嵌套层数没有限制。在生产环境中，我们建议嵌套层数不超过 3 层。

- 关于外部目录异步物化视图：

  - 外部目录物化视图仅支持异步固定间隔刷新和手动刷新。
  - 不能保证物化视图与外部目录中的基表之间的严格一致性。
  - 目前不支持基于外部资源构建物化视图。
  - 目前 StarRocks 无法感知外部目录的基表数据是否发生了变化，因此每次刷新基表时，默认都会刷新所有分区。您只能使用 [REFRESH MATERIALIZED VIEW 手动刷新部分分区](../data-manipulation/REFRESH_MATERIALIZED_VIEW.md)。

## 例子

### 同步物化视图的示例

基表的结构如下：

```Plain Text
mysql> desc duplicate_table;
+-------+--------+------+------+---------+-------+
| Field | Type   | Null | Key  | Default | Extra |
+-------+--------+------+------+---------+-------+
| k1    | INT    | Yes  | true | N/A     |       |
| k2    | INT    | Yes  | true | N/A     |       |
| k3    | BIGINT | Yes  | true | N/A     |       |
| k4    | BIGINT | Yes  | true | N/A     |       |
+-------+--------+------+------+---------+-------+
```

示例 1：创建仅包含原始表（k1、k2）列的同步物化视图。

```sql
create materialized view k1_k2 as
select k1, k2 from duplicate_table;
```

物化视图仅包含两列 k1 和 k2，没有任何聚合。

```plain text
+-----------------+-------+--------+------+------+---------+-------+
| IndexName       | Field | Type   | Null | Key  | Default | Extra |
+-----------------+-------+--------+------+------+---------+-------+
| k1_k2           | k1    | INT    | Yes  | true | N/A     |       |
|                 | k2    | INT    | Yes  | true | N/A     |       |
+-----------------+-------+--------+------+------+---------+-------+
```

示例 2：创建按 k2 排序的同步物化视图。

```sql
create materialized view k2_order as
select k2, k1 from duplicate_table order by k2;
```

物化视图的架构如下所示。物化视图仅包含两列 k2 和 k1，其中列 k2 是没有任何聚合的排序列。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| IndexName       | Field | Type   | Null | Key   | Default | Extra |
+-----------------+-------+--------+------+-------+---------+-------+
| k2_order        | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k1    | INT    | Yes  | false | N/A     | NONE  |
+-----------------+-------+--------+------+-------+---------+-------+
```

示例 3：创建按 k1 和 k2 分组的同步物化视图，并在 k3 上创建 SUM 聚合。

```sql
create materialized view k1_k2_sumk3 as
select k1, k2, sum(k3) from duplicate_table group by k1, k2;
```

物化视图的架构如下所示。物化视图包含三列 k1、k2 和 sum （k3），其中 k1、k2 是分组列，sum （k3） 是根据 k1 和 k2 分组的 k3 列的总和。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| IndexName       | Field | Type   | Null | Key   | Default | Extra |
+-----------------+-------+--------+------+-------+---------+-------+
| k1_k2_sumk3     | k1    | INT    | Yes  | true  | N/A     |       |
|                 | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k3    | BIGINT | Yes  | false | N/A     | SUM   |
+-----------------+-------+--------+------+-------+---------+-------+
```

因为物化视图没有声明排序列，并且采用了聚合功能，因此 StarRocks 默认对分组列 k1 和 k2 进行补充。

示例 4：创建同步物化视图以删除重复行。

```sql
create materialized view deduplicate as
select k1, k2, k3, k4 from duplicate_table group by k1, k2, k3, k4;
```

物化视图的架构如下所示。物化视图包含 k1、k2、k3 和 k4 列，并且没有重复的行。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| IndexName       | Field | Type   | Null | Key   | Default | Extra |
+-----------------+-------+--------+------+-------+---------+-------+
| deduplicate     | k1    | INT    | Yes  | true  | N/A     |       |
|                 | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k3    | BIGINT | Yes  | true  | N/A     |       |
|                 | k4    | BIGINT | Yes  | true  | N/A     |       |
+-----------------+-------+--------+------+-------+---------+-------+
```

示例 5：创建不声明排序列的非聚合同步物化视图。

基表的结构如下图所示：

```plain text
+-------+--------------+------+-------+---------+-------+
| Field | Type         | Null | Key   | Default | Extra |
+-------+--------------+------+-------+---------+-------+
| k1    | TINYINT      | Yes  | true  | N/A     |       |
| k2    | SMALLINT     | Yes  | true  | N/A     |       |
| k3    | INT          | Yes  | true  | N/A     |       |
| k4    | BIGINT       | Yes  | true  | N/A     |       |
| k5    | DECIMAL(9,0) | Yes  | true  | N/A     |       |
| k6    | DOUBLE       | Yes  | false | N/A     | NONE  |
| k7    | VARCHAR(20)  | Yes  | false | N/A     | NONE  |
+-------+--------------+------+-------+---------+-------+
```

物化视图包含 k3、k4、k5、k6 和 k7 列，并且未声明排序列。使用以下语句创建物化视图：

```sql
create materialized view mv_1 as
select k3, k4, k5, k6, k7 from all_type_table;
```

StarRocks 默认自动使用 k3、k4 和 k5 作为排序列。这三种列类型占用的字节数之和为 4 （INT） + 8 （BIGINT） + 16 （DECIMAL） = 28 < 36。因此，这三列被添加为排序列。

物化视图的架构如下所示。

```plain text
+----------------+-------+--------------+------+-------+---------+-------+
| IndexName      | Field | Type         | Null | Key   | Default | Extra |
+----------------+-------+--------------+------+-------+---------+-------+
| mv_1           | k3    | INT          | Yes  | true  | N/A     |       |
|                | k4    | BIGINT       | Yes  | true  | N/A     |       |
|                | k5    | DECIMAL(9,0) | Yes  | true  | N/A     |       |
|                | k6    | DOUBLE       | Yes  | false | N/A     | NONE  |
|                | k7    | VARCHAR(20)  | Yes  | false | N/A     | NONE  |
+----------------+-------+--------------+------+-------+---------+-------+
```

可以看出，`key`k3、k4 和 k5 列的字段是 `true`，表示它们是排序键。k6 和 k7 列的键字段是 `false`，表示它们不是排序键。

示例 6：创建包含 WHERE 子句和复杂表达式的同步物化视图。

```SQL
-- 创建基表：user_event
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

  -- 使用 WHERE 子句和复杂表达式创建物化视图。
  CREATE MATERIALIZED VIEW test_mv1
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

### 异步物化视图示例

以下示例基于以下基表：

```SQL
CREATE TABLE `lineorder` (
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
COMMENT "OLAP"
PARTITION BY RANGE(`lo_orderdate`)
(PARTITION p1 VALUES [("-2147483648"), ("19930101")),
PARTITION p2 VALUES [("19930101"), ("19940101")),
PARTITION p3 VALUES [("19940101"), ("19950101")),
PARTITION p4 VALUES [("19950101"), ("19960101")),
PARTITION p5 VALUES [("19960101"), ("19970101")),
PARTITION p6 VALUES [("19970101"), ("19980101")),
PARTITION p7 VALUES [("19980101"), ("19990101")))
DISTRIBUTED BY HASH(`lo_orderkey`);

CREATE TABLE IF NOT EXISTS `customer` (
  `c_custkey` int(11) NOT NULL COMMENT "",
  `c_name` varchar(26) NOT NULL COMMENT "",
  `c_address` varchar(41) NOT NULL COMMENT "",
  `c_city` varchar(11) NOT NULL COMMENT "",
  `c_nation` varchar(16) NOT NULL COMMENT "",
  `c_region` varchar(13) NOT NULL COMMENT "",
  `c_phone` varchar(16) NOT NULL COMMENT "",
  `c_mktsegment` varchar(11) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`c_custkey`)
COMMENT "OLAP"
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
COMMENT "OLAP"
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
COMMENT "OLAP"
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
COMMENT "OLAP"
DISTRIBUTED BY HASH(`p_partkey`);

create table orders ( 
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
) 
PRIMARY KEY (dt, order_id) 
PARTITION BY RANGE(`dt`) 
( PARTITION p20210820 VALUES [('2021-08-20'), ('2021-08-21')), 
PARTITION p20210821 VALUES [('2021-08-21'), ('2021-08-22')) ) 
DISTRIBUTED BY HASH(order_id)
PROPERTIES (
    "replication_num" = "3", 
    "enable_persistent_index" = "true"
);
```

示例 1：创建非分区化的物化视图。

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

示例 2：创建分区化的物化视图。

```SQL
CREATE MATERIALIZED VIEW lo_mv2
PARTITION BY `lo_orderdate`
DISTRIBUTED BY HASH(`lo_orderkey`)
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

-- 使用 date_trunc() 函数按月份对物化视图进行分区。
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

示例 3：创建异步物化视图。

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

示例 4：创建分区化的物化视图，并使用 `str2date` 将基表的 STRING 类型分区键转换为物化视图的日期类型。

``` SQL

-- 具有字符串分区列的 Hive 表。
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


-- 使用 `str2date` 创建物化视图。
CREATE MATERIALIZED VIEW IF NOT EXISTS `test_mv` 
按str2date(`d_datekey`,'%Y%m%d')进行分区
按HASH(`d_date`, `d_month`, `d_month`)进行分布
手动刷新
作为
选择
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
从
 `hive_catalog`.`ssb_1g_orc`.`part_dates` ;