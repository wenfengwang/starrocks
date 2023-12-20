---
displayed_sidebar: English
---

# 聚合表

当您创建一个使用聚合表的表时，可以定义排序键列和度量列，并为度量列指定聚合函数。如果待加载的记录具有相同的排序键，则度量列会被聚合。聚合表有助于减少查询过程中需要处理的数据量，从而加快查询速度。

## 应用场景

聚合表非常适用于数据统计和分析场景。以下是几个例子：

- 帮助网站或应用程序提供商分析用户在特定网站或应用上的流量和停留时间，以及网站或应用的总访问次数。

- 帮助广告代理分析其为客户提供的广告的总点击次数、总浏览量和消费统计。

- 帮助电商公司分析其年度交易数据，识别每个季度或月份内的地理热销商品。

上述场景中的数据查询和摄入具有以下特征：

- 大部分查询都是聚合查询，如 SUM、MAX 和 MIN。
- 无需检索原始详细数据。
- 历史数据不经常更新，只追加新数据。

## 原理

从数据摄入到数据查询，使用聚合表的表中具有相同排序键的数据将进行多次聚合，具体如下：

1. 在数据摄入阶段，当数据批量加载到表中，每个批次包括一个数据版本。数据版本生成后，StarRocks 会聚合该数据版本中具有相同排序键的数据。
2. 在后台压缩阶段，当数据摄入产生的多个数据版本的文件被定期压缩成一个大文件时，StarRocks 会聚合这个大文件中具有相同排序键的数据。
3. 在数据查询阶段，StarRocks 会在返回查询结果之前，聚合所有数据版本中具有相同排序键的数据。

聚合操作有助于减少需要处理的数据量，从而加快查询速度。

假设您有一个使用聚合表的表，并想将以下四条原始记录加载到表中。

|日期|国家/地区|PV|
|---|---|---|
|2020.05.01|中文|1|
|2020.05.01|中文|2|
|2020.05.01|美国|3|
|2020.05.01|美国|4|

在数据摄入时，StarRocks 会将这四条原始记录聚合成以下两条记录。

|日期|国家/地区|PV|
|---|---|---|
|2020.05.01|中文|3|
|2020.05.01|美国|7|

## 创建表

假设您想分析不同城市的用户访问不同网页的次数。在这个示例中，创建一个名为 example_db.aggregate_tbl 的表，将 site_id、date 和 city_code 定义为排序键列，将 pv 定义为度量列，并为 pv 列指定 SUM 函数。

创建表的语句如下：

```SQL
CREATE TABLE IF NOT EXISTS example_db.aggregate_tbl (
    site_id LARGEINT NOT NULL COMMENT "id of site",
    date DATE NOT NULL COMMENT "time of event",
    city_code VARCHAR(20) COMMENT "city_code of user",
    pv BIGINT SUM DEFAULT "0" COMMENT "total page views"
)
AGGREGATE KEY(site_id, date, city_code)
DISTRIBUTED BY HASH(site_id)
PROPERTIES (
"replication_num" = "3"
);
```

> **注意**
- 创建表时，您必须使用 `DISTRIBUTED BY HASH` 子句指定桶列。详细信息请参阅[bucketing](../Data_distribution.md#design-partitioning-and-bucketing-rules)文档。
- 从 v2.5.7 版本开始，StarRocks 在创建表或添加分区时可以自动设置桶（BUCKETS）的数量，无需手动设置桶的数量。详细信息请参阅[determine the number of buckets](../Data_distribution.md#determine-the-number-of-buckets)。

## 使用须知

- 关于表的排序键，需要注意以下几点：
-   您可以使用 AGGREGATE KEY 关键字显式定义排序键中使用的列。

    - 如果 AGGREGATE KEY 关键字没有包含所有维度列，表将无法创建。
    - 默认情况下，如果您没有使用 AGGREGATE KEY 关键字显式定义排序键列，StarRocks 会将除度量列之外的所有列作为排序键列。

-   排序键必须在强制执行唯一性约束的列上创建，且必须包含所有维度列，这些列的名称不能更改。

- 您可以在列名之后指定聚合函数，以将该列定义为度量列。通常情况下，度量列用于保存需要聚合和分析的数据。

- 关于聚合表支持的聚合函数的信息，请参见[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)文档。

- 执行查询时，排序键列在多数据版本聚合之前进行筛选，而度量列在聚合之后进行筛选。因此，我们建议您确定哪些列经常用作筛选条件，并将这些列定义为排序键，这样可以在多数据版本聚合之前开始数据筛选，以提高查询性能。

- 创建表时，不能在表的度量列上创建 BITMAP 索引或布隆过滤器索引。

## 下一步

创建表后，您可以使用多种数据摄入方法将数据加载到 StarRocks。有关 StarRocks 支持的数据摄入方法的信息，请参见[数据导入](../../loading/Loading_intro.md)文档。

> 注意：当您将数据加载到使用聚合表的表时，只能更新表的所有列。例如，更新之前提到的 example_db.aggregate_tbl 表时，您必须更新其所有列，包括 site_id、date、city_code 和 pv。
