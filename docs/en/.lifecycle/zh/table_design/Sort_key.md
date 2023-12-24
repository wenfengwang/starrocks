---
displayed_sidebar: English
---

# 排序键和前缀索引

创建表时，可以选择其中一个或多个列来组成排序键。排序键确定数据存储在磁盘上之前表中数据的排序顺序。您可以将排序键列用作查询的筛选条件。因此，StarRocks 可以快速定位感兴趣的数据，而无需扫描整个表以查找需要处理的数据。这降低了搜索复杂性，从而加快了查询速度。

此外，为了减少内存消耗，StarRocks 支持在表上创建前缀索引。前缀索引是一种备用索引。StarRocks 将表的每 1024 行存储在一个块中，为其生成一个索引条目并存储在前缀索引表中。块的前缀索引条目长度不能超过 36 个字节，其内容是由该块第一行中表的排序键列组成的前缀。这有助于 StarRocks 在搜索前缀索引表时，快速定位到存储该行数据的区块的起始列号。表的前缀索引在大小上比表本身小 1024 倍。因此，可以将整个前缀索引缓存在内存中，以帮助加速查询。

## 原则

在“重复键”表中，使用 `DUPLICATE KEY` 关键字定义排序键列。

在聚合表中，排序键列是使用 `AGGREGATE KEY` 关键字定义的。

在“唯一键”表中，使用 `UNIQUE KEY` 关键字定义排序键列。

从 v3.0 开始，主键和排序键在主键表中解耦。排序键列是使用 `ORDER BY` 关键字定义的。主键列是使用 `PRIMARY KEY` 关键字定义的。

在为“重复键”表、“聚合”表或“唯一键”表定义排序键列时，请注意以下几点：

- 排序键列必须是连续定义的列，其中第一个定义的列必须是起始排序键列。

- 计划选择作为排序键列的列必须在其他常用列之前定义。

- 列出排序键列的顺序必须符合定义表列的顺序。

以下示例显示了由四列组成的表中允许的排序键列和不允许的排序键列，分别是 `site_id`、 `city_code`、 `user_id` 和 `pv`：

- 允许的排序键列的示例
  - `site_id` 和 `city_code`
  - `site_id`、 `city_code` 和 `user_id`

- 不允许的排序键列的示例
  - `city_code` 和 `site_id`
  - `city_code` 和 `user_id`
  - `site_id`、 `city_code` 和 `pv`

以下各节提供了在创建不同类型的表时如何定义排序键列的示例。这些示例适用于至少具有 3 个 BE 的 StarRocks 集群。

### 重复键

创建一个名为 `site_access_duplicate` 的表。该表由四列组成：`site_id`、`city_code`、`user_id` 和 `pv`，其中 `site_id` 和 `city_code` 被选为排序键列。

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
>
> 从 v2.5.7 开始，StarRocks 可以在创建表或添加分区时自动设置 BUCKET 数量。您不再需要手动设置存储桶数量。有关详细信息，请参阅 [确定存储桶数量](./Data_distribution.md#determine-the-number-of-buckets)。

### 聚合键

创建一个名为 `site_access_aggregate` 的表。该表由四列组成：`site_id`、`city_code`、`user_id` 和 `pv`，其中 `site_id` 和 `city_code` 被选为排序键列。

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

>**注意**
>
> 对于聚合表，未指定的列为键列，指定的列为值列。请参阅 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。在前面的示例中，只有 `site_id` 和 `city_code` 被指定为排序键列，因此 `agg_type` 必须为 `user_id` 和 `pv` 指定。

### 唯一键

创建一个名为 `site_access_unique` 的表。该表由四列组成：`site_id`、`city_code`、`user_id` 和 `pv`，其中 `site_id` 和 `city_code` 被选为排序键列。

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

### 主键

创建一个名为 `site_access_primary` 的表。该表由四列组成：`site_id`、`city_code`、`user_id` 和 `pv`，其中 `site_id` 被选为主键列，`site_id` 和 `city_code` 被选为排序键列。

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
ORDER BY(site_id,city_code);
```

## 排序效果

以上表为例。排序效果在以下三种情况下有所不同：

- 如果查询同时筛选 `site_id` 和 `city_code`，则 StarRocks 在查询过程中需要扫描的行数会显著减少：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123 and city_code = 2;
  ```

- 如果您的查询只在 `site_id` 上进行筛选，StarRocks 可以将查询范围缩小到包含 `site_id` 值的行：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123;
  ```

- 如果您的查询只在 `city_code` 上进行筛选，StarRocks 需要扫描整个表：

  ```Plain
  select sum(pv) from site_access_duplicate where city_code = 2;
  ```

  > **注意**
  >
  > 在此情况下，排序键列不会产生预期的排序效果。

如上所述，当您的查询同时筛选 `site_id` 和 `city_code` 时，StarRocks 会对该表进行二进制搜索，将查询范围缩小到特定位置。如果表包含大量行，StarRocks 会改为对 `site_id` 和 `city_code` 列进行二进制搜索。这需要 StarRocks 将两列的数据加载到内存中，从而增加内存消耗。在这种情况下，您可以使用前缀索引来减少内存中缓存的数据量，从而加快查询速度。

此外，请注意，大量的排序键列也会增加内存消耗。为了减少内存消耗，StarRocks 对前缀索引的使用设置了以下限制：

- 块的前缀索引条目必须由该块第一行中表的排序键列的前缀组成。

- 最多可以在 3 列上创建前缀索引。

- 前缀索引条目的长度不能超过 36 个字节。

- 不能在 FLOAT 或 DOUBLE 数据类型的列上创建前缀索引。

- 在创建前缀索引的所有列中，只允许有一列 VARCHAR 数据类型，并且该列必须是前缀索引的结束列。

- 如果前缀索引的结束列是 CHAR 或 VARCHAR 数据类型，则前缀索引中的任何条目都不能超过 36 个字节。

## 如何选择排序键列

本节以 `site_access_duplicate` 表为例，介绍如何选择排序键列。

- 我们建议您确定查询经常筛选的列，并选择这些列作为排序键列。

- 如果选择多个排序键列，建议先列出经常筛选的高鉴别级别列，然后再列出其他列。
  
  如果列中的值数较大且持续增长，则该列具有较高的鉴别级别。例如，表中的城市数是固定的`site_access_duplicate`，这意味着表列中的值数是固定`city_code`的。但是，列中的值数`site_id`远大于列中的值数，`city_code`并且会不断增长。因此，`site_id`色谱柱的鉴别水平高于色谱柱 `city_code` 。

- 我们建议您不要选择大量的排序键列。大量的排序键列不能帮助提高查询性能，但会增加排序和数据加载的开销。

总之，在为 `site_access_duplicate` 表选择排序键列时，请注意以下几点：

- 如果您的查询经常同时筛选 `site_id` 和 `city_code`，我们建议您选择 `site_id` 作为开始排序键列。

- 如果您的查询经常只筛选 `city_code`，偶尔筛选 `site_id` 和 `city_code`，我们建议您选择 `city_code` 作为开始排序键列。

- 如果查询对 `site_id` 和 `city_code` 进行筛选的次数大致等于查询仅筛选 `city_code` 的次数，建议您创建一个物化视图，其第一列为 `city_code`。因此，StarRocks 会在 `city_code` 物化视图的列上创建一个排序索引。
