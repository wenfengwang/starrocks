---
displayed_sidebar: English
---

# 使用位图进行精确去重计数

本主题介绍如何在StarRocks中使用位图来计算不同值的数量。

位图是计算数组中不同值数量的有效工具。与传统的去重计数相比，这种方法占用更少的存储空间，并且能够加快计算速度。假设存在一个名为A的数组，其值的范围是[0, n)。通过使用(n+7)/8字节的位图，你可以计算出数组中不同元素的数量。为此，首先将所有位初始化为0，然后将元素的值作为位的索引，并将对应的位设置为1。位图中1的个数即为数组中不同元素的数量。

## 传统去重计数

StarRocks采用MPP架构，使用Count Distinct功能时能够保留详细数据。然而，Count Distinct特性在查询处理过程中需要进行多次数据洗牌操作，这会消耗更多资源，并且随着数据量的增长，性能会线性下降。

以下场景是基于表(dt, page, user_id)中的详细数据来计算UV的例子。

|dt|页面|用户 ID|
|---|---|---|
|20191206|游戏|101|
|20191206|购物|102|
|20191206|游戏|101|
|20191206|购物|101|
|20191206|游戏|101|
|20191206|购物|101|

StarRocks会根据下图所示进行数据计算。首先，它会根据page和user_id列对数据进行分组，然后计数处理后的结果。

![alter](../assets/6.1.2-2.png)

* 注意：图示展示了在两个BE节点上计算的6行数据的概念图。

当处理大量需要多次洗牌操作的数据时，所需的计算资源可能会显著增加，这会导致查询速度变慢。然而，使用位图技术可以帮助解决这个问题，并在这类场景下提高查询性能。

按页面分组计数UV：

```sql
select page, count(distinct user_id) as uv from table group by page;

|  page   |   uv  |
| :---: | :---: |
|   game  |  1   |
|   shopping  |   2  |
```

## 使用位图进行去重计数的好处

与COUNT(DISTINCT expr)相比，使用位图可以带来以下好处：

* 更少的存储空间：如果使用位图来计算INT32数据的不同值数量，所需的存储空间只有COUNT(DISTINCT expr)的1/32。StarRocks使用压缩的Roaring位图来执行计算，与传统位图相比，进一步减少了存储空间的占用。
* 更快的计算速度：位图使用位运算，计算速度比COUNT(DISTINCT expr)更快。在StarRocks中，不同值数量的计算可以并行处理，从而进一步提升查询性能。

有关Roaring位图的实现，请参见[specific paper and implementation](https://github.com/RoaringBitmap/RoaringBitmap)。

## 使用须知

* 位图索引和位图去重计数都使用位图技术。然而，它们的引入目的和解决的问题完全不同。前者用于过滤低基数的枚举列，而后者用于计算数据行的值列中不同元素的数量。
* StarRocks 2.3及更高版本支持将值列定义为BITMAP类型，不论表的类型（聚合表、重复键表、主键表或唯一键表）。然而，表的[排序键](../table_design/Sort_key.md)不能是BITMAP类型。
* 创建表时，你可以将值列定义为**BITMAP**，并将聚合函数定义为[**BITMAP_UNION**](../sql-reference/sql-functions/bitmap-functions/bitmap_union.md)。
* 你只能使用Roaring位图来计算以下类型数据的不同值数量：TINYINT、SMALLINT、INT和BIGINT。对于其他类型的数据，则需要[构建全局字典](#global-dictionary)。

## 使用位图进行去重计数

以页面UV计算为例：

1. 创建一个包含BITMAP列visit_users的聚合表，使用BITMAP_UNION作为聚合函数。

   ```sql
   CREATE TABLE `page_uv` (
     `page_id` INT NOT NULL COMMENT 'page ID',
     `visit_date` datetime NOT NULL COMMENT 'access time',
     `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'user ID'
   ) ENGINE=OLAP
   AGGREGATE KEY(`page_id`, `visit_date`)
   DISTRIBUTED BY HASH(`page_id`)
   PROPERTIES (
     "replication_num" = "3",
     "storage_format" = "DEFAULT"
   );
   ```

2. 将数据加载到此表中。

   使用INSERT INTO来加载数据：

   ```sql
   INSERT INTO page_uv VALUES
   (1, '2020-06-23 01:30:30', to_bitmap(13)),
   (1, '2020-06-23 01:30:30', to_bitmap(23)),
   (1, '2020-06-23 01:30:30', to_bitmap(33)),
   (1, '2020-06-23 02:30:30', to_bitmap(13)),
   (2, '2020-06-23 01:30:30', to_bitmap(23));
   ```

   数据加载后：

   *  在page_id = 1、visit_date = '2020-06-23 01:30:30'的行中，visit_users字段包含三个位图元素(13、23、33)。
   *  在page_id = 1、visit_date = '2020-06-23 02:30:30'的行中，visit_users字段包含一个位图元素(13)。
   *  在page_id = 2、visit_date = '2020-06-23 01:30:30'的行中，visit_users字段包含一个位图元素(23)。

   从本地文件加载数据：

   ```shell
   echo -e '1,2020-06-23 01:30:30,130\n1,2020-06-23 01:30:30,230\n1,2020-06-23 01:30:30,120\n1,2020-06-23 02:30:30,133\n2,2020-06-23 01:30:30,234' > tmp.csv | 
   curl --location-trusted -u <username>:<password> -H "label:label_1600960288798" \
       -H "column_separator:," \
       -H "columns:page_id,visit_date,visit_users, visit_users=to_bitmap(visit_users)" -T tmp.csv \
       http://StarRocks_be0:8040/api/db0/page_uv/_stream_load
   ```

3. 计算页面UV。

   ```sql
   SELECT page_id, count(distinct visit_users) FROM page_uv GROUP BY page_id;
   +-----------+------------------------------+
   |  page_id  | count(DISTINCT `visit_users`)|
   +-----------+------------------------------+
   |         1 |                            3 |
   |         2 |                            1 |
   +-----------+------------------------------+
   2 row in set (0.00 sec)
   ```

## 全局字典

目前，基于位图的去重计数机制要求输入为整数类型。如果用户需要使用其他数据类型作为位图的输入，则需要构建自己的全局字典，将其他类型的数据（如字符串类型）映射为整数类型。构建全局字典有多种方法：

### 基于Hive表的全局字典

在这种方案中，全局字典本身是一个Hive表，包含两列：原始值和编码后的整数值。生成全局字典的步骤如下：

1. 对事实表中的字典列进行去重，生成一个临时表。
2. 将临时表与全局字典进行左连接，向临时表中添加新值。
3. 对新值进行编码，并将其插入到全局字典中。
4. 将事实表与更新后的全局字典进行左连接，用ID替换字典项。

通过这种方式，可以使用Spark或MR更新全局字典并替换事实表中的值列。与基于trie树的全局字典相比，这种方法可以分布式执行，并且全局字典可以重复使用。

但是，需要注意的是：原始事实表被多次读取，并且在全局字典计算过程中有两次连接操作，这会消耗大量额外资源。

### 基于trie树构建全局字典

用户也可以使用trie树（也称为前缀树或字典树）来构建自己的全局字典。trie树的节点后代具有共同的前缀，这有助于减少查询时间并最小化字符串比较，因此非常适合实现字典编码。然而，trie树的实现不易分布式处理，在数据量较大时可能会成为性能瓶颈。

通过构建全局字典并将其他类型的数据转换为整数数据，可以利用位图对非整数数据列进行精确的去重计数分析。
