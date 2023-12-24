---
displayed_sidebar: English
---

# 聚合表

创建使用聚合表的表时，您可以定义排序键列和指标列，并可以为指标列指定聚合函数。如果要加载的记录具有相同的排序键，则会对指标列进行聚合。聚合表有助于减少需要处理的数据量，从而加快查询速度。

## 场景

聚合表非常适合数据统计和分析方案。下面有几个例子：

- 帮助网站或应用提供商分析用户在特定网站或应用上花费的流量和时间，以及访问网站或应用的总次数。

- 帮助广告代理商分析他们为客户提供的广告的总点击次数、总观看次数和消费统计数据。

- 帮助电子商务公司分析其年度交易数据，以确定各个季度或月份内的地理畅销商品。

上述场景下的数据查询和引入具有以下特点：

- 大多数查询是聚合查询，例如 SUM、MAX 和 MIN。
- 不需要检索原始详细数据。
- 历史数据不经常更新。仅追加新数据。

## 原则

从数据引入到数据查询，使用聚合表的表中具有相同排序键的数据将按如下方式进行多次聚合：

1. 在数据引入阶段，当数据作为批次加载到表中时，每个批次都包含一个数据版本。生成数据版本后，StarRocks 会聚合数据版本中具有相同排序键的数据。
2. 在后台压缩阶段，当数据引入时生成的多个数据版本的文件周期性地压缩成一个大文件时，StarRocks 会将具有相同排序键的数据聚合到大文件中。
3. 在数据查询阶段，StarRocks 会将所有数据版本中排序键相同的数据进行聚合，然后返回查询结果。

聚合操作有助于减少需要处理的数据量，从而加快查询速度。

假设您有一个使用聚合表的表，并希望将以下四条原始记录加载到该表中。

| 日期       | 国家 | PV   |
| ---------- | ------- | ---- |
| 2020.05.01 | 中国     | 1    |
| 2020.05.01 | 中国     | 2    |
| 2020.05.01 | 美国     | 3    |
| 2020.05.01 | 美国     | 4    |

StarRocks 在数据引入时，会将 4 条原始记录聚合为以下 2 条记录。

| 日期       | 国家 | PV   |
| ---------- | ------- | ---- |
| 2020.05.01 | 中国     | 3    |
| 2020.05.01 | 美国     | 7    |

## 创建表

假设您要分析不同城市的用户对不同网页的访问次数。在此示例中，创建一个名为 `example_db.aggregate_tbl` 的表，定义 `site_id`、 `date`和 `city_code` 作为排序键列，定义 `pv` 为指标列，并为该列指定 SUM 函数。

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
>
> - 创建表时，必须使用 `DISTRIBUTED BY HASH` 子句指定存储桶列。有关详细信息，请参阅 [存储桶](../Data_distribution.md#design-partitioning-and-bucketing-rules)。
> - 从 v2.5.7 开始，StarRocks 可以在您创建表或添加分区时自动设置存储桶数量 (BUCKETS)。您不再需要手动设置存储桶数量。有关详细信息，请参阅 [确定存储桶数量](../Data_distribution.md#determine-the-number-of-buckets)。

## 使用说明

- 关于表的排序键，请注意以下几点：
  - 您可以使用 `AGGREGATE KEY` 关键字显式定义排序键中使用的列。

    - 如果 `AGGREGATE KEY` 关键字未包含所有维度列，则无法创建表。
    - 默认情况下，如果您没有使用 `AGGREGATE KEY` 关键字显式定义排序键列，StarRocks 会选择除指标列以外的所有列作为排序键列。

  - 排序键必须在强制执行唯一约束的列上创建。它必须由名称无法更改的所有维度列组成。

- 您可以在列名称后指定聚合函数，以将该列定义为指标列。在大多数情况下，指标列包含需要聚合和分析的数据。

- 有关聚合表支持的聚合函数的信息，请参阅 [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

- 运行查询时，排序键列在聚合多个数据版本之前进行筛选，而指标列在聚合多个数据版本之后进行筛选。因此，建议您确定经常用作筛选条件的列，并将这些列定义为排序键。这样，数据筛选可以在聚合多个数据版本之前开始，以提高查询性能。

- 创建表时，不能对表的指标列创建 BITMAP 索引或 Bloom Filter 索引。

## 下一步做什么

表创建成功后，您可以使用多种数据引入方式将数据加载到 StarRocks 中。有关 StarRocks 支持的数据引入方式的信息，请参见 [数据导入](../../loading/Loading_intro.md)。

> 注意：将数据加载到使用聚合表的表中时，只能更新表的所有列。例如，更新上述 `example_db.aggregate_tbl` 表时，必须更新其所有列，即 `site_id`、 `date`、 `city_code` 和 `pv`。
