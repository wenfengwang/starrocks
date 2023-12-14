---
displayed_sidebar: "Chinese"
---

# 使用位图进行精确计数去重

本主题描述了如何在 StarRocks 中使用位图来计算去重值的数量。

位图是一个用于计算数组中不同值数量的有用工具。与传统的计数去重相比，这种方法占用的存储空间更少，并且可以加速计算。假设有一个名为 A 的数组，其取值范围为 [0, n)。通过使用 (n+7)/8 字节的位图，您可以计算数组中不同元素的数量。只需初始化所有位为 0，将元素值设置为位的下标，然后将所有位设置为 1。位图中的 1 的数量即为数组中不同元素的数量。

## 传统的计数去重

StarRocks 使用 MPP 架构，可以在使用计数去重时保留详细数据。但是，计数去重功能在查询处理过程中需要多次数据洗牌，这会消耗更多资源，并导致性能线性下降。

下面的示例基于表 (dt, page, user_id) 中的详细数据计算 UVs。

|  dt   |   page  | user_id |
| :---: | :---: | :---:|
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 102 |
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 101 |
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 101 |

StarRocks 根据以下图形计算数据。它首先通过 `page` 和 `user_id` 列对数据进行分组，然后计算处理后的结果。

![alter](../assets/6.1.2-2.png)

*注意：该图显示了在两个 BE 节点上计算的 6 行数据的示意图。

当处理需要多次洗牌操作的大容量数据时，所需的计算资源可能会显著增加。这会减慢查询速度。但是，在这种情况下，使用位图技术可以帮助解决这个问题，并改善查询性能。

按 `page` 进行 UV 计数：

```sql
select page, count(distinct user_id) as uv from table group by page;

|  page   |   uv  |
| :---: | :---: |
|   game  |  1   |
|   shopping  |   2  |
```

## 使用位图进行计数去重的优势

与 COUNT(DISTINCT expr) 相比，您可以从以下方面通过位图获得益处：

* 存储空间更少：如果使用位图来计算 INT32 数据的不同值数量，则所需的存储空间仅为 COUNT(DISTINCT expr) 的 1/32。StarRocks 利用压缩的 roaring 位图执行计算，与传统位图相比，进一步减少了存储空间的使用。
* 计算更快：位图使用位操作，导致比 COUNT(DISTINCT expr) 更快的计算。在 StarRocks 中，不同值的数量的计算可以并行处理，进一步改进了查询性能。

有关 Roaring Bitmap 的实现，请参见[特定论文和实现](https://github.com/RoaringBitmap/RoaringBitmap)。

## 使用注意事项

* 位图索引和位图计数去重都使用了位图技术。但是，引入它们的目的和解决的问题是完全不同的。前者用于以低基数过滤枚举列，而后者用于计算数据行的值列中不同元素的数量。
* StarRocks 2.3 及更高版本支持将值列定义为 BITMAP，而不管表类型是（聚合表、重复键表、主键表还是唯一键表）。但是，表的[排序键](../table_design/Sort_key.md)不能是 BITMAP 类型。
* 创建表时，可以将值列定义为 BITMAP，并将聚合函数定义为[BITMAP_UNION](../sql-reference/sql-functions/bitmap-functions/bitmap_union.md)。
* 您只能使用 roaring 位图来计算 TINYINT、SMALLINT、INT 和 BIGINT 数据的不同值数量。对于其他类型的数据，您需要[构建全局字典](#global-dictionary)。

## 使用位图进行计数去重

以计算页面 UV 为例。

1. 创建一个具有 BITMAP 列 `visit_users`（使用聚合函数 BITMAP_UNION）的聚合表。

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

2. 将数据加载到此表中。

    使用 INSET INTO 加载数据：

    ```sql
    INSERT INTO page_uv VALUES
    (1, '2020-06-23 01:30:30', to_bitmap(13)),
    (1, '2020-06-23 01:30:30', to_bitmap(23)),
    (1, '2020-06-23 01:30:30', to_bitmap(33)),
    (1, '2020-06-23 02:30:30', to_bitmap(13)),
    (2, '2020-06-23 01:30:30', to_bitmap(23));
    ```

    数据加载后：

    * 在行 `page_id = 1, visit_date = '2020-06-23 01:30:30'` 中，`visit_users` 字段包含三个位图元素（13, 23, 33）。
    * 在行 `page_id = 1, visit_date = '2020-06-23 02:30:30'` 中，`visit_users` 字段包含一个位图元素（13）。
    * 在行 `page_id = 2, visit_date = '2020-06-23 01:30:30'` 中，`visit_users` 字段包含一个位图元素（23）。

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
    2 行 (0.00 秒)
    ```

## 全局字典

当前，基于位图的计数去重机制要求输入为整数。如果用户需要将其他数据类型用于位图的计数去重，则需要构建自己的全局字典来将其他类型的数据（例如字符串类型）映射为整数类型。有几种构建全局字典的思路。

### 基于 Hive 表的全局字典

在这种方案中，全局字典本身是一个 Hive 表，它有两列，一列用于原始值，另一列用于编码的整数值。生成全局字典的步骤如下：

1. 对事实表的字典列进行去重，生成临时表
2. 将临时表与全局字典左连接，向临时表添加 `new value`
3. 对 `new value` 进行编码，并插入到全局字典中
4. 将事实表和更新后的全局字典左连接，替换字典项为 ID

通过这种方式，全局字典可以更新，并且可以使用 Spark 或 MR 替换事实表中的值列。与基于 trie 树的全局字典相比，这种方法可以分布式并且全局字典可被重用。

然而，需要注意以下几点：原始事实表会被多次读取，并且在计算全局字典时会有两次连接，消耗了很多额外资源。

### 基于 trie 树的全局字典的构建

用户还可以使用Trie树（也称为前缀树或字典树）构建自己的全局词典。Trie树具有节点的后代的共同前缀，可以用来减少查询时间和最小化字符串比较，因此非常适合实现字典编码。然而，Trie树的实现不容易分发，并且在数据量相对较大时可能会产生性能瓶颈。

通过构建全局词典并将其他类型的数据转换为整数数据，可以使用位图对非整数数据列执行准确的不重复计数分析。