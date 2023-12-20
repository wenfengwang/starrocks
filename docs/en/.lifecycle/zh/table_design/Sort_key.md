---
displayed_sidebar: English
---

# 排序键和前缀索引

当您创建表时，可以选择一个或多个列作为排序键。排序键决定了表数据在存储到磁盘之前的排序顺序。您可以在查询中使用排序键列作为过滤条件。这样，StarRocks 可以快速定位到感兴趣的数据，无需扫描整个表来找到需要处理的数据。这降低了搜索的复杂性，因而加快了查询速度。

此外，为了减少内存消耗，StarRocks 支持在表上创建前缀索引。前缀索引是一种稀疏索引。StarRocks 将表的每 1024 行数据存储在一个块中，并为此生成一个索引条目，存储在前缀索引表中。块的前缀索引条目长度不能超过 36 字节，其内容是该块第一行中表的排序键列组成的前缀。这有助于 StarRocks 在搜索前缀索引表时快速定位到存储该行数据的块的起始列号。表的前缀索引大小是表本身大小的 1/1024。因此，整个前缀索引可以被缓存在内存中，帮助加速查询。

## 原则

在 Duplicate Key 表中，排序键列使用 `DUPLICATE KEY` 关键字定义。

在 Aggregate 表中，排序键列使用 `AGGREGATE KEY` 关键字定义。

在 Unique Key 表中，排序键列使用 `UNIQUE KEY` 关键字定义。

从 v3.0 版本开始，Primary Key 表中的主键和排序键是解耦的。排序键列使用 `ORDER BY` 关键字定义，主键列使用 `PRIMARY KEY` 关键字定义。

为 Duplicate Key 表、Aggregate 表或 Unique Key 表定义排序键列时，请注意以下几点：

- 排序键列必须是连续定义的列，第一个定义的列必须是起始排序键列。

- 您计划选择作为排序键列的列必须在其他普通列之前定义。

- 您列出排序键列的顺序必须遵循定义表列的顺序。

以下示例展示了由 `site_id`、`city_code`、`user_id` 和 `pv` 这四列组成的表中允许和不允许的排序键列：

- 允许的排序键列示例
  - `site_id` 和 `city_code`
  - `site_id`、`city_code` 和 `user_id`

- 不允许的排序键列示例
  - `city_code` 和 `site_id`
  - `city_code` 和 `user_id`
  - `site_id`、`city_code` 和 `pv`

以下部分提供了在创建不同类型表时如何定义排序键列的示例。这些示例适用于至少有三个 BE 的 StarRocks 集群。

### Duplicate Key

创建一个名为 `site_access_duplicate` 的表。该表包括四列：`site_id`、`city_code`、`user_id` 和 `pv`，其中 `site_id` 和 `city_code` 被选为排序键列。

创建表的语句如下：

```SQL
CREATE TABLE site_access_duplicate
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_id INT,
    pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(site_id, city_code)
DISTRIBUTED BY HASH(site_id);
```

> **注意**
> 从 v2.5.7 版本开始，StarRocks 在创建表或添加分区时可以自动设置桶数（BUCKETS）。您不再需要手动设置桶数。详细信息请参见[确定桶数](./Data_distribution.md#determine-the-number-of-buckets)。

### Aggregate Key

创建一个名为 `site_access_aggregate` 的表。该表包括四列：`site_id`、`city_code`、`user_id` 和 `pv`，其中 `site_id` 和 `city_code` 被选为排序键列。

创建表的语句如下：

```SQL
CREATE TABLE site_access_aggregate
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_id BITMAP BITMAP_UNION,
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(site_id, city_code)
DISTRIBUTED BY HASH(site_id);
```

> **注意**
> 对于 Aggregate 表，未指定 `agg_type` 的列为键列，指定了 `agg_type` 的列为值列。详见 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。在上述示例中，仅 `site_id` 和 `city_code` 被指定为排序键列，因此必须为 `user_id` 和 `pv` 指定 `agg_type`。

### Unique Key

创建一个名为 `site_access_unique` 的表。该表包括四列：`site_id`、`city_code`、`user_id` 和 `pv`，其中 `site_id` 和 `city_code` 被选为排序键列。

创建表的语句如下：

```SQL
CREATE TABLE site_access_unique
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_id INT,
    pv BIGINT DEFAULT '0'
)
UNIQUE KEY(site_id, city_code)
DISTRIBUTED BY HASH(site_id);
```

### Primary Key

创建一个名为 `site_access_primary` 的表。该表包括四列：`site_id`、`city_code`、`user_id` 和 `pv`，其中 `site_id` 被选为主键列，`site_id` 和 `city_code` 被选为排序键列。

创建表的语句如下：

```SQL
CREATE TABLE site_access_primary
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_id INT,
    pv BIGINT DEFAULT '0'
)
PRIMARY KEY(site_id)
DISTRIBUTED BY HASH(site_id)
ORDER BY(site_id, city_code);
```

## 排序效果

以前述表为例，以下三种情况下排序效果有所不同：

- 如果您的查询同时过滤 `site_id` 和 `city_code`，StarRocks 在查询期间需要扫描的行数将显著减少：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123 and city_code = 2;
  ```

- 如果您的查询只过滤 `site_id`，StarRocks 可以将查询范围缩小到包含 `site_id` 值的行：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123;
  ```

- 如果您的查询只过滤 `city_code`，StarRocks 需要扫描整个表：

  ```Plain
  select sum(pv) from site_access_duplicate where city_code = 2;
  ```

    > **注**
    > 在这种情况下，排序键列不会产生预期的排序效果。

如上所述，当您的查询同时过滤 `site_id` 和 `city_code` 时，StarRocks 会在表上执行二分搜索，将查询范围缩小到特定位置。如果表包含大量行，StarRocks 会对 `site_id` 和 `city_code` 列执行二分搜索。这要求 StarRocks 将这两列的数据加载到内存中，从而增加内存消耗。在这种情况下，您可以使用前缀索引来减少缓存在内存中的数据量，从而加速查询。

此外，注意大量排序键列也会增加内存消耗。为了减少内存消耗，StarRocks 对前缀索引的使用施加了以下限制：

- 块的前缀索引条目必须由该块第一行的表排序键列的前缀组成。

- 最多可以在三列上创建前缀索引。

- 前缀索引条目长度不能超过 36 字节。

- 不能在 FLOAT 或 DOUBLE 数据类型的列上创建前缀索引。

- 在所有创建前缀索引的列中，只允许有一列是 VARCHAR 数据类型，并且该列必须是前缀索引的最后一列。

- 如果前缀索引的最后一列是 CHAR 或 VARCHAR 数据类型，则前缀索引中的任何条目都不能超过 36 字节。

## 如何选择排序键列

本节以 `site_access_duplicate` 表为例，介绍如何选择排序键列。

- 我们建议您识别出查询中经常用作过滤条件的列，并将这些列作为排序键列。

- 如果您选择了多个排序键列，我们建议您将区分度较高的列排在前面。

  如果列的值数量大且持续增长，则该列具有较高的区分度。例如，`site_access_duplicate` 表中的城市数量是固定的，这意味着 `city_code` 列的值数量是固定的。然而，`site_id` 列的值数量远大于 `city_code` 列，并且持续增长。因此，`site_id` 列的区分度高于 `city_code` 列。

- 我们建议您不要选择过多的排序键列。过多的排序键列不仅不能提高查询性能，反而会增加排序和数据加载的开销。

总结来说，为 `site_access_duplicate` 表选择排序键列时，请注意以下几点：

- 如果您的查询经常同时过滤 `site_id` 和 `city_code`，我们建议您将 `site_id` 作为起始排序键列。

- 如果您的查询经常只过滤 `city_code`，偶尔同时过滤 `site_id` 和 `city_code`，我们建议您将 `city_code` 作为起始排序键列。

- 如果您的查询过滤 `site_id` 和 `city_code` 的次数大致相等，我们建议您创建一个物化视图，其第一列为 `city_code`。这样，StarRocks 会在物化视图的 `city_code` 列上创建排序索引。