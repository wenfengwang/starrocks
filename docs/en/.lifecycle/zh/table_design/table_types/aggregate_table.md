---
displayed_sidebar: English
---

# 聚合表

当您创建一个使用聚合表的表时，可以定义排序键列和度量列，并为度量列指定聚合函数。如果待加载的记录具有相同的排序键，则度量列将被聚合。聚合表有助于减少查询过程中需要处理的数据量，从而加快查询速度。

## 应用场景

聚合表非常适合于数据统计和分析场景。以下是一些示例：

- 帮助网站或应用程序提供商分析用户在特定网站或应用上的流量和停留时间，以及网站或应用的总访问次数。

- 帮助广告代理商分析他们为客户提供的广告的总点击数、总浏览量和消费统计。

- 帮助电商公司分析其年度交易数据，以识别每个季度或月份内的地理热销商品。

上述场景中的数据查询和导入具有以下特征：

- 大部分查询都是聚合查询，如 SUM、MAX 和 MIN。
- 无需检索原始详细数据。
- 历史数据不经常更新，只追加新数据。

## 原理

从数据导入到数据查询，使用聚合表的表中具有相同排序键的数据会经过多次聚合，具体如下：

1. 在数据导入阶段，当数据以批次形式加载到表中时，每个批次包含一个数据版本。数据版本生成后，StarRocks 会对该数据版本中具有相同排序键的数据进行聚合。
2. 在后台压缩阶段，当数据导入时生成的多个数据版本的文件定期合并成一个大文件时，StarRocks 会对大文件中具有相同排序键的数据进行聚合。
3. 在数据查询阶段，StarRocks 会在返回查询结果之前，聚合所有数据版本中具有相同排序键的数据。

聚合操作有助于减少需要处理的数据量，从而加快查询速度。

假设您有一个使用聚合表的表，并想要将以下四条原始记录加载到表中。

|日期|国家|PV|
|---|---|---|
|2020.05.01|CHN|1|
|2020.05.01|CHN|2|
|2020.05.01|USA|3|
|2020.05.01|USA|4|

StarRocks 在数据导入时将这四条原始记录聚合成以下两条记录。

|日期|国家|PV|
|---|---|---|
|2020.05.01|CHN|3|
|2020.05.01|USA|7|

## 创建表

假设您想分析不同城市的用户访问不同网页的次数。在此示例中，创建一个名为 `example_db.aggregate_tbl` 的表，定义 `site_id`、`date` 和 `city_code` 为排序键列，定义 `pv` 为度量列，并为 `pv` 列指定 SUM 聚合函数。

创建表的语句如下：

```SQL
CREATE TABLE IF NOT EXISTS example_db.aggregate_tbl (
    site_id LARGEINT NOT NULL COMMENT "site id",
    date DATE NOT NULL COMMENT "event time",
    city_code VARCHAR(20) COMMENT "user city code",
    pv BIGINT SUM DEFAULT "0" COMMENT "total page views"
)
AGGREGATE KEY(site_id, date, city_code)
DISTRIBUTED BY HASH(site_id)
PROPERTIES (
"replication_num" = "3"
);
```

> **注意**
- 创建表时，您必须使用 `DISTRIBUTED BY HASH` 子句来指定分桶列。有关详细信息，请参见[bucketing](../Data_distribution.md#design-partitioning-and-bucketing-rules)。
- 从 v2.5.7 版本开始，StarRocks 在创建表或添加分区时可以自动设置桶（BUCKETS）的数量，无需手动设置。有关详细信息，请参见[determine the number of buckets](../Data_distribution.md#determine-the-number-of-buckets)。

## 使用说明

- 关于表的排序键，请注意以下几点：
-   您可以使用 `AGGREGATE KEY` 关键字来显式定义排序键中使用的列。

    - 如果 `AGGREGATE KEY` 关键字没有包含所有维度列，表将无法创建。
    - 默认情况下，如果您没有使用 `AGGREGATE KEY` 关键字显式定义排序键列，StarRocks 会选择除度量列之外的所有列作为排序键列。

-   排序键必须在强制执行唯一性约束的列上创建，并且必须包含所有不可更改名称的维度列。

- 您可以在列名后指定一个聚合函数，以将该列定义为度量列。通常，度量列用于保存需要聚合和分析的数据。

- 有关聚合表支持的聚合函数信息，请参见 [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

- 运行查询时，排序键列在多个数据版本聚合之前进行过滤，而度量列在聚合之后进行过滤。因此，我们建议您确定经常用作过滤条件的列，并将这些列定义为排序键，这样可以在多个数据版本聚合之前开始数据过滤，以提升查询性能。

- 创建表时，不能在表的度量列上创建 BITMAP 索引或 Bloom Filter 索引。

## 下一步

创建表后，您可以使用多种数据导入方法将数据加载到 StarRocks 中。有关 StarRocks 支持的数据导入方法的信息，请参见[数据导入](../../loading/Loading_intro.md)。

> 注意：当您向使用聚合表的表中加载数据时，只能更新表的所有列。例如，当您更新前述 `example_db.aggregate_tbl` 表时，必须更新其所有列，包括 `site_id`、`date`、`city_code` 和 `pv`。