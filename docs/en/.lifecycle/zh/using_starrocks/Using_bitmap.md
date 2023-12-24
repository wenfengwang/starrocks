---
displayed_sidebar: English
---

# 使用位图进行精确的计数去重

本主题描述了如何在StarRocks中使用位图来计算不同值的数量。

位图是计算数组中不同值数量的有用工具。与传统的Count Distinct相比，这种方法占用的存储空间更少，并且可以加速计算。假设有一个名为A的数组，其取值范围为[0，n)。通过使用(n+7)/8字节的位图，您可以计算数组中不同元素的数量。为此，将所有位初始化为0，将元素的值设置为位的下标，然后将所有位设置为1。位图中1的数量就是数组中不同元素的数量。

## 传统的Count Distinct

StarRocks使用MPP架构，在使用Count Distinct时可以保留详细数据。但是，Count Distinct功能在查询处理过程中需要多次数据洗牌，这会消耗更多资源，并导致性能随着数据量的增加而线性下降。

以下场景根据表（dt、page、user_id）中的详细数据计算UVs。

|  dt   |   page  | user_id |
| :---: | :---: | :---:|
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 102 |
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 101 |
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 101 |

StarRocks根据下图计算数据。它首先按`page`和`user_id`列对数据进行分组，然后对处理后的结果进行计数。

![alter](../assets/6.1.2-2.png)

* 注意：该图显示了在两个BE节点上计算的6行数据的示意图。

在处理大量需要多次洗牌操作的数据时，所需的计算资源可能会显著增加。这会减慢查询速度。但是，在这种情况下，使用位图技术可以帮助解决此问题，并提高查询性能。

按`page`分组计算`uv`：

```sql
select page, count(distinct user_id) as uv from table group by page;

|  page   |   uv  |
| :---: | :---: |
|   game  |  1   |
|   shopping  |   2  |
```

## 使用位图进行Count Distinct的优势

与COUNT(DISTINCT expr)相比，您可以从位图中获得以下几个方面的优势：

* 存储空间更少：如果使用位图计算INT32数据的不同值数量，则所需的存储空间仅为COUNT(DISTINCT expr)的1/32。StarRocks利用压缩的Roaring位图来执行计算，与传统位图相比，进一步减少了存储空间的使用。
* 更快的计算：位图使用位操作，计算速度比COUNT(DISTINCT expr)更快。在StarRocks中，不同值数量的计算可以并行处理，从而进一步提高查询性能。

有关Roaring Bitmap的实现，请参见[具体论文和实现](https://github.com/RoaringBitmap/RoaringBitmap)。

## 使用说明

* 位图索引和位图Count Distinct都使用位图技术。但是，介绍它们的目的和它们解决的问题完全不同。前者用于筛选基数较低的枚举列，而后者用于计算数据行的值列中不同元素的数量。
* StarRocks 2.3及更高版本支持将值列定义为BITMAP，无论表类型如何（聚合表、重复键表、主键表或唯一键表）。但是，表的[排序键](../table_design/Sort_key.md)不能是BITMAP类型。
* 创建表时，可以将值列定义为BITMAP，将聚合函数定义为[BITMAP_UNION](../sql-reference/sql-functions/bitmap-functions/bitmap_union.md)。
* 您只能使用Roaring位图来计算以下类型的数据的不同值数量：TINYINT、SMALLINT、INT和BIGINT。对于其他类型的数据，需要[构建全局字典](#global-dictionary)。

## 使用位图计算Count Distinct

以计算页面UVs为例。

1. 创建一个带有BITMAP列`visit_users`的聚合表，该表使用聚合函数BITMAP_UNION。

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

    使用INSET INTO加载数据：

    ```sql
    INSERT INTO page_uv VALUES
    (1, '2020-06-23 01:30:30', to_bitmap(13)),
    (1, '2020-06-23 01:30:30', to_bitmap(23)),
    (1, '2020-06-23 01:30:30', to_bitmap(33)),
    (1, '2020-06-23 02:30:30', to_bitmap(13)),
    (2, '2020-06-23 01:30:30', to_bitmap(23));
    ```

    加载数据后：

    * 在行`page_id = 1, visit_date = '2020-06-23 01:30:30'`中，`visit_users`字段包含三个位图元素（13、23、33）。
    * 在行`page_id = 1, visit_date = '2020-06-23 02:30:30'`中，`visit_users`字段包含一个位图元素（13）。
    * 在行`page_id = 2, visit_date = '2020-06-23 01:30:30'`中，`visit_users`字段包含一个位图元素（23）。

   从本地文件加载数据：

    ```shell
    echo -e '1,2020-06-23 01:30:30,130\n1,2020-06-23 01:30:30,230\n1,2020-06-23 01:30:30,120\n1,2020-06-23 02:30:30,133\n2,2020-06-23 01:30:30,234' > tmp.csv | 
    curl --location-trusted -u <username>:<password> -H "label:label_1600960288798" \
        -H "column_separator:," \
        -H "columns:page_id,visit_date,visit_users, visit_users=to_bitmap(visit_users)" -T tmp.csv \
        http://StarRocks_be0:8040/api/db0/page_uv/_stream_load
    ```

3. 计算页面UVs。

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

目前，基于位图的Count Distinct机制要求输入为整数。如果用户需要使用其他数据类型作为位图的输入，则用户需要构建自己的全局字典，以将其他类型的数据（如字符串类型）映射到整数类型。有几种构建全局字典的想法。

### 基于Hive表的全局字典

此方案中的全局字典本身是一个Hive表，它有两列，一列用于原始值，一列用于编码的Int值。生成全局字典的步骤如下：

1. 对事实表的字典列进行去重，生成临时表
2. 左连接临时表和全局字典，将`new value`添加到临时表中。
3. 对`new value`进行编码，并将其插入全局字典中。
4. 左连接事实表和更新后的全局字典，用ID替换字典项。

这样，就可以更新全局字典，并使用Spark或MR替换事实表中的值列。与基于trie树的全局字典相比，这种方法可以分布式，全局字典可以重复使用。

但是，有几点需要注意：原始事实表被多次读取，并且在计算全局字典时有两个连接，会消耗大量额外资源。

### 基于trie树构建全局字典

用户还可以使用trie树（也称为前缀树或字典树）构建自己的全局字典。trie树具有节点后代的通用前缀，可用于减少查询时间并最小化字符串比较，因此非常适合实现字典编码。但是，trie树的实现不容易分布，并且在数据量相对较大时可能会造成性能瓶颈。

通过构建全局字典并将其他类型的数据转换为整数数据，您可以使用位图对非整数数据列执行准确的Count Distinct分析。
