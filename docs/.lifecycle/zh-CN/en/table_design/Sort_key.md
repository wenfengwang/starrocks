---
displayed_sidebar: "English"
---

# 排序键和前缀索引

在创建表时，可以选择其中一个或多个列组成一个排序键。排序键确定了数据存储到磁盘前的排序顺序。您可以将排序键列用作查询的筛选条件。因此，StarRocks能够快速定位感兴趣的数据，无需扫描整个表以查找需要处理的数据。这降低了查询的复杂度，加快了查询速度。

此外，为了减少内存消耗，StarRocks支持在表上创建前缀索引。前缀索引是一种稀疏索引。StarRocks将表的每1024行存储为一个块，为该块生成一个索引条目，并将其存储在前缀索引表中。一个块的前缀索引条目长度不能超过36个字节，其内容是该块首行中由表的排序键列组成的前缀。这有助于StarRocks在运行对前缀索引表的搜索时快速定位存储该行数据的块的起始列号。表的前缀索引大小是表本身的1024分之一。因此，整个前缀索引可以缓存在内存中，以帮助加快查询速度。

## 原则

在**重复键**表中，使用`DUPLICATE KEY`关键字定义排序键列。

在**聚合**表中，使用`AGGREGATE KEY`关键字定义排序键列。

在**唯一键**表中，使用`UNIQUE KEY`关键字定义排序键列。

从v3.0开始，在**主键**表中，主键和排序键被解耦。排序键列使用`ORDER BY`关键字定义，主键列使用`PRIMARY KEY`关键字定义。

在为**重复键**表、**聚合**表或**唯一键**表定义排序键列时，请注意以下几点：

- 排序键列必须是连续定义的列，其中第一个定义的列必须是起始排序键列。

- 您计划选择为排序键列的列必须在其他普通列之前进行定义。

- 您列出排序键列的顺序必须符合表的列定义顺序。

以下示例展示了由4列组成的表的允许和不允许的排序键列：

- 允许的排序键列示例
  - `site_id`和`city_code`
  - `site_id`、`city_code`和`user_id`

- 不允许的排序键列示例
  - `city_code`和`site_id`
  - `city_code`和`user_id`
  - `site_id`、`city_code`和`pv`

以下部分提供了在创建不同类型的表时如何定义排序键列的示例。这些示例适用于至少有三个BE的StarRocks集群。

### 重复键

创建一个名为`site_access_duplicate`的表。该表由四列组成：`site_id`、`city_code`、`user_id`和`pv`，其中`site_id`和`city_code`被选择为排序键列。

创建表的语句如下:

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
>
> 从v2.5.7开始，当您创建表或添加分区时，StarRocks可以自动设置存储桶的数量（BUCKETS）。您无需手动设置存储桶数量。有关详细信息，请参见 [确定存储桶的数量](./Data_distribution.md#确定存储桶的数量)。

### 聚合键

创建一个名为`site_access_aggregate`的表。该表由四列组成：`site_id`、`city_code`、`user_id`和`pv`，其中`site_id`和`city_code`被选择为排序键列。

创建表的语句如下:

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

>**注意**
>
> 对于聚合表，未指定`agg_type`的列是键列，指定了`agg_type`的列是值列。请参见 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。在上面的示例中，只有`site_id`和`city_code`被指定为排序键列，因此`agg_type`必须指定为`user_id`和`pv`。

### 唯一键

创建一个名为`site_access_unique`的表。该表由四列组成：`site_id`、`city_code`、`user_id`和`pv`，其中`site_id`和`city_code`被选择为排序键列。

创建表的语句如下:

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

### 主键

创建一个名为`site_access_primary`的表。该表由四列组成：`site_id`、`city_code`、`user_id`和`pv`，其中`site_id`被选择为主键列，`site_id`和`city_code`被选择为排序键列。

创建表的语句如下:

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
ORDER BY(site_id,city_code);
```

## 排序效果

以上述表格为例，排序效果在以下三种情况下有所不同:

- 如果您的查询同时过滤`site_id`和`city_code`，StarRocks在查询过程中需要扫描的行数将显著减少:

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123 and city_code = 2;
  ```

- 如果您的查询仅过滤`site_id`，StarRocks可以将查询范围缩小到包含`site_id`值的行:

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123;
  ```

- 如果您的查询仅过滤`city_code`，StarRocks需要扫描整个表:

  ```Plain
  select sum(pv) from site_access_duplicate where city_code = 2;
  ```

  > **注意**
  >
  > 在此情况下，排序键列未产生预期的排序效果。

如上所述，当查询同时过滤`site_id`和`city_code`时，StarRocks在表上运行二分搜索来将查询范围缩小到特定位置。如果表包含大量行，则StarRocks将在`site_id`和`city_code`列上运行二分搜索。这要求StarRocks将两个列的数据加载到内存中，从而增加了内存消耗。在这种情况下，您可以使用前缀索引来减少内存中缓存的数据量，从而加快查询速度。

此外，大量的排序键列也会增加内存消耗。为了减少内存消耗，StarRocks对前缀索引的使用施加了以下限制：

- 一个块的前缀索引条目必须由该块首行的表的排序键列前缀组成。

- 一个前缀索引最多可以在3列上创建。

- 前缀索引条目的长度不能超过36个字节。

- 不能在FLOAT或DOUBLE数据类型的列上创建前缀索引。

- 如果为前缀索引创建的列中有一个是VARCHAR数据类型，那么该列必须是前缀索引的结尾列。

- 如果前缀索引的结尾列是CHAR或VARCHAR数据类型，则前缀索引中的条目不能超过36个字节。

## 如何选择排序键列

本节以`site_access_duplicate`表为例，介绍如何选择排序键列。

- 我们建议您确定经常使用作为查询筛选条件的列，并将这些列选为排序键列。

- 如果选择多个排序键列，我们建议您将高区分级别的常过滤列列出在其他列之前。
一个列具有很高的区分水平，如果列中的值的数量很大并且不断增长。例如，`site_access_duplicate` 表中城市的数量是固定的，这意味着表中 `city_code` 列的值的数量是固定的。然而，`site_id` 列中的值的数量要比 `city_code` 列中的值的数量大得多，并且不断增长。因此，`site_id` 列比 `city_code` 列具有更高的区分水平。

- 我们建议您不要选择大量的分类关键列。大量的分类关键列无法帮助提高查询性能，反而会增加排序和数据加载的开销。

总之，在为 `site_access_duplicate` 表选择分类关键列时，请注意以下几点：

- 如果您的查询经常同时过滤 `site_id` 和 `city_code`，我们建议您选择 `site_id` 作为开始的分类关键列。

- 如果您的查询经常只过滤 `city_code`，偶尔过滤 `site_id` 和 `city_code`，我们建议您选择 `city_code` 作为开始的分类关键列。

- 如果您的查询中过滤 `site_id` 和 `city_code` 的次数大致等于只过滤 `city_code` 的次数，我们建议您创建一个物化视图，其中第一列是 `city_code`。这样，StarRocks 会在物化视图的 `city_code` 列上创建一个排序索引。