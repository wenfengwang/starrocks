---
displayed_sidebar: English
---

# 使用 Bitmap 进行精确去重计数

本主题介绍如何使用 Bitmap 来计算 StarRocks 中唯一值的数量。

Bitmap 是计算数组中唯一值数量的有用工具。与传统的 Count Distinct 相比，该方法占用的存储空间更少，并且可以加速计算。假设有一个名为 A 的数组，其取值范围为 [0, n)。通过使用 (n+7)/8 字节的 Bitmap，您可以计算数组中唯一元素的数量。为此，将所有位初始化为 0，将元素的值设置为位的下标，然后将所有位设置为 1。Bitmap 中 1 的数量就是数组中唯一元素的数量。

## 传统 Count Distinct

StarRocks 采用 MPP 架构，在使用 Count Distinct 时可以保留详细数据。但 Count Distinct 特性在查询处理过程中需要进行多次数据 shuffle，这会消耗更多的资源，并且随着数据量的增加，性能会线性下降。

以下场景根据表（dt、page、user_id）中的详细数据计算 UV。

|dt|page|user_id|
|---|---|---|
|20191206|game|101|
|20191206|shopping|102|
|20191206|game|101|
|20191206|shopping|101|
|20191206|game|101|
|20191206|shopping|101|

StarRocks 按照下图计算数据。它首先按 `page` 和 `user_id` 列对数据进行分组，然后对处理结果进行计数。

![alter](../assets/6.1.2-2.png)

* 注：该图显示了在两个 BE 节点上计算的 6 行数据的示意图。

当处理需要多次 shuffle 操作的大量数据时，所需的计算资源可能会显著增加。这会减慢查询速度。然而，使用 Bitmap 技术可以帮助解决这个问题并提高此类场景下的查询性能。

按 `page` 分组计算 `uv`：

```sql
select page, count(distinct user_id) as uv from table group by page;

|  page   |   uv  |
| :---: | :---: |
|   game  |  1   |
|   shopping  |   2  |
```

## 使用 Bitmap 进行 Count Distinct 的好处

与 COUNT(DISTINCT expr) 相比，您可以在以下方面从 Bitmap 中受益：

* 更少的存储空间：如果使用 Bitmap 来计算 INT32 数据的唯一值数量，所需的存储空间仅为 COUNT(DISTINCT expr) 的 1/32。StarRocks 利用压缩的 Roaring Bitmap 来执行计算，与传统 Bitmap 相比，进一步减少了存储空间的使用。
* 计算速度更快：Bitmap 使用按位运算，与 COUNT(DISTINCT expr) 相比，计算速度更快。在 StarRocks 中，唯一值数量的计算可以并行处理，从而进一步提高查询性能。

Roaring Bitmap 的实现参见[具体论文和实现](https://github.com/RoaringBitmap/RoaringBitmap)。

## 使用说明

* 位图索引和位图 Count Distinct 都使用 Bitmap 技术。但引入它们的目的和解决的问题却完全不同。前者用于过滤基数较低的枚举列，后者用于计算数据行的值列中唯一元素的数量。
* StarRocks 2.3 及更高版本支持将值列定义为 BITMAP，无论表类型如何（聚合表、Duplicate Key 表、Primary Key 表或 Unique Key 表）。但表的[排序键](../table_design/Sort_key.md)不能是 BITMAP 类型。
* 创建表时，可以将值列定义为 BITMAP，并将聚合函数定义为 [BITMAP_UNION](../sql-reference/sql-functions/bitmap-functions/bitmap_union.md)。
* 您只能使用 Roaring Bitmap 来计算以下类型数据的唯一值数量：TINYINT、SMALLINT、INT 和 BIGINT。对于其他类型的数据，需要[构建全局字典](#global-dictionary)。

## 使用 Bitmap 进行 Count Distinct

以页面 UV 的计算为例。

1. 创建一个包含 BITMAP 列 `visit_users` 的聚合表，该表使用聚合函数 BITMAP_UNION。

   ```sql
   CREATE TABLE `page_uv` (
     `page_id` INT NOT NULL COMMENT '页面 ID',
     `visit_date` datetime NOT NULL COMMENT '访问时间',
     `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT '用户 ID'
   ) ENGINE=OLAP
   AGGREGATE KEY(`page_id`, `visit_date`)
   DISTRIBUTED BY HASH(`page_id`)
   PROPERTIES (
     "replication_num" = "3",
     "storage_format" = "DEFAULT"
   );
   ```

2. 将数据加载到该表中。

   使用 INSERT INTO 加载数据：

   ```sql
   INSERT INTO page_uv VALUES
   (1, '2020-06-23 01:30:30', to_bitmap(13)),
   (1, '2020-06-23 01:30:30', to_bitmap(23)),
   (1, '2020-06-23 01:30:30', to_bitmap(33)),
   (1, '2020-06-23 02:30:30', to_bitmap(13)),
   (2, '2020-06-23 01:30:30', to_bitmap(23));
   ```

   数据加载后：

   * 在 `page_id = 1, visit_date = '2020-06-23 01:30:30'` 行中，`visit_users` 字段包含三个 Bitmap 元素 (13、23、33)。
   * 在 `page_id = 1, visit_date = '2020-06-23 02:30:30'` 行中，`visit_users` 字段包含一个 Bitmap 元素 (13)。
   * 在 `page_id = 2, visit_date = '2020-06-23 01:30:30'` 行中，`visit_users` 字段包含一个 Bitmap 元素 (23)。

   从本地文件加载数据：

   ```shell
   echo -e '1,2020-06-23 01:30:30,130\n1,2020-06-23 01:30:30,230\n1,2020-06-23 01:30:30,120\n1,2020-06-23 02:30:30,133\n2,2020-06-23 01:30:30,234' > tmp.csv | 
   curl --location-trusted -u <username>:<password> -H "label:label_1600960288798" \
       -H "column_separator:," \
       -H "columns:page_id,visit_date,visit_users, visit_users=to_bitmap(visit_users)" -T tmp.csv \
       http://StarRocks_be0:8040/api/db0/page_uv/_stream_load
   ```

3. 计算页面 UV。

   ```sql
   SELECT page_id, count(distinct visit_users) FROM page_uv GROUP BY page_id;
   +-----------+------------------------------+
   |  page_id  | count(DISTINCT `visit_users`)|
   +-----------+------------------------------+
   |         1 |                            3 |
   |         2 |                            1 |
   +-----------+------------------------------+
   2 rows in set (0.00 sec)
   ```

## 全局字典

目前，基于 Bitmap 的 Count Distinct 机制要求输入为整数。如果用户需要使用其他数据类型作为 Bitmap 的输入，则需要构建自己的全局字典，将其他类型的数据（例如字符串类型）映射到整数类型。构建全局字典有多种方法。

### 基于 Hive 表的全局字典

在此方案中，全局字典本身是一个 Hive 表，它有两列，一列用于原始值，一列用于编码的 Int 值。生成全局字典的步骤如下：

1. 对事实表的字典列进行去重，生成临时表。
2. 将临时表与全局字典进行左连接，向临时表添加 `new value`。
3. 对 `new value` 进行编码并将其插入全局字典中。
4. 将事实表与更新后的全局字典进行左连接，用 ID 替换字典项。

这样，可以使用 Spark 或 MR 更新全局字典并替换事实表中的值列。与基于 trie 树的全局字典相比，此方法可以分布式执行，并且全局字典可以重用。

但是，需要注意的是：原始事实表被多次读取，并且在全局字典的计算过程中存在两次连接操作，这会消耗大量额外资源。

### 基于 trie 树构建全局字典

用户还可以使用 trie 树（又称前缀树或字典树）构建自己的全局字典。trie 树对于节点的后代有共同的前缀，可以用来减少查询时间并最小化字符串比较，因此非常适合实现字典编码。然而，trie 树的实现不易分布式执行，在数据量较大时可能会产生性能瓶颈。

通过构建全局字典，并将其他类型的数据转换为整数数据，可以使用 Bitmap 对非整数数据列进行精确的 Count Distinct 分析。