---
displayed_sidebar: English
---

# 排序键和前缀索引

在创建表时，您可以选择一列或多列作为排序键。排序键决定了表中数据在存储到磁盘前的排序顺序。您可以将排序键列作为查询的过滤条件，这样StarRocks可以快速定位到感兴趣的数据，避免扫描整个表来寻找需要处理的数据。这样做降低了搜索的复杂度，因而加速了查询。

此外，为了降低内存消耗，StarRocks支持在表上创建前缀索引。前缀索引是一种稀疏索引。StarRocks将表中的每1024行数据存储在一个块中，并为这个块生成一个索引条目，存储在前缀索引表中。块的前缀索引条目的长度不能超过36字节，其内容是该块第一行中表的排序键列所组成的前缀。这有助于StarRocks在查询前缀索引表时快速定位到存储该行数据的块的起始列号。表的前缀索引大小是表本身的1/1024，因此整个前缀索引可以缓存在内存中，帮助加速查询。

## 原则

在重复键表中，排序键列使用DUPLICATE KEY关键字定义。

在聚合表中，排序键列使用AGGREGATE KEY关键字定义。

在唯一键表中，排序键列使用UNIQUE KEY关键字定义。

从v3.0版本开始，主键表的主键和排序键解耦。排序键列用ORDER BY关键字定义，主键列用PRIMARY KEY关键字定义。

在为重复键表、聚合表或唯一键表定义排序键列时，请注意以下几点：

- 排序键列必须是连续定义的列，第一个定义的列必须是起始排序键列。

- 您计划选作排序键列的列必须在其他普通列之前定义。

- 列出排序键列的顺序必须遵循定义表列的顺序。

以下示例展示了由四列组成的表（site_id、city_code、user_id和pv）中允许和不允许的排序键列：

- 允许的排序键列示例：
  - site_id和city_code
  - site_id、city_code和user_id

- 不允许的排序键列示例：
  - city_code和site_id
  - city_code和user_id
  - site_id、city_code和pv

以下部分提供了在创建不同类型表时如何定义排序键列的示例。这些示例适用于至少有三个BE节点的StarRocks集群。

### 重复键表

创建一个名为site_access_duplicate的表。该表包含四列：site_id、city_code、user_id和pv，其中site_id和city_code被选作排序键列。

创建该表的语句如下：

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
> 从v2.5.7版本开始，StarRocks在创建表或添加分区时可以自动设置桶（**BUCKETS**）的数量，无需手动设置桶数。详情请参阅[确定桶的数量](./Data_distribution.md#determine-the-number-of-buckets)。

### 聚合键表

创建一个名为site_access_aggregate的表。该表包含四列：site_id、city_code、user_id和pv，其中site_id和city_code被选作排序键列。

创建该表的语句如下：

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
> 对于聚合表，未指定`agg_type`的列是键列，指定了`agg_type`的列是值列。详情请参阅[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。在上述示例中，只有`site_id`和`city_code`被指定为排序键列，因此必须为`user_id`和`pv`指定`agg_type`。

### 唯一键表

创建一个名为site_access_unique的表。该表包含四列：site_id、city_code、user_id和pv，其中site_id和city_code被选作排序键列。

创建该表的语句如下：

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

### 主键表

创建一个名为site_access_primary的表。该表包含四列：site_id、city_code、user_id和pv，其中site_id被选作主键列，site_id和city_code被选作排序键列。

创建该表的语句如下：

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

以前述表为例，根据以下三种情况，排序效果各不相同：

- 如果您的查询同时过滤site_id和city_code，则StarRocks在查询时需要扫描的行数会显著减少：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123 and city_code = 2;
  ```

- 如果您的查询仅过滤site_id，StarRocks可以将查询范围缩小到包含特定site_id值的行：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123;
  ```

- 如果您的查询仅过滤city_code，StarRocks需要扫描整个表：

  ```Plain
  select sum(pv) from site_access_duplicate where city_code = 2;
  ```

    > **注意**
    > 在这种情况下，排序键列不会产生预期的排序效果。

如前所述，当您的查询同时过滤site_id和city_code时，StarRocks会在表上执行二分搜索，将查询范围缩小到特定位置。如果表包含大量行，StarRocks会对site_id和city_code列执行二分搜索。这要求StarRocks将这两列的数据加载到内存中，因此会增加内存消耗。在这种情况下，您可以使用前缀索引来减少缓存到内存中的数据量，从而加速查询。

另外，请注意，大量的排序键列也会增加内存消耗。为了降低内存消耗，StarRocks对前缀索引的使用施加了以下限制：

- 块的前缀索引条目必须由该块第一行中表的排序键列前缀组成。

- 最多可以在三列上创建前缀索引。

- 前缀索引条目的长度不能超过36字节。

- 不能在FLOAT或DOUBLE数据类型的列上创建前缀索引。

- 在所有创建了前缀索引的列中，只允许有一列是VARCHAR数据类型，并且该列必须是前缀索引的最后一列。

- 如果前缀索引的最后一列是CHAR或VARCHAR数据类型，前缀索引中的任何条目都不能超过36字节。

## 如何选择排序键列

本节以site_access_duplicate表为例，介绍如何选择排序键列。

- 我们建议您识别出查询中经常使用的过滤列，并将这些列作为排序键列。

- 如果您选择了多个排序键列，我们建议您将区分度较高的频繁过滤列放在其他列之前。

  列的区分度高是指该列的值数量庞大且持续增长。例如，site_access_duplicate表中城市的数量是固定的，这意味着city_code列的值数量是固定的。然而，site_id列的值数量远大于city_code列，并且持续增长。因此，site_id列的区分度高于city_code列。

- 我们建议您不要选择过多的排序键列。大量排序键列不仅无助于提高查询性能，反而会增加排序和数据加载的开销。

总之，在为site_access_duplicate表选择排序键列时，请牢记以下几点：

- 如果您的查询经常同时过滤site_id和city_code，我们建议您将site_id作为起始排序键列。

- 如果您的查询经常只过滤city_code，偶尔同时过滤site_id和city_code，我们建议您将city_code作为起始排序键列。

- 如果您的查询中过滤site_id和city_code的次数大致等同于只过滤city_code的次数，我们建议您创建一个物化视图，其第一列为city_code。这样，StarRocks会在物化视图的city_code列上创建排序索引。
