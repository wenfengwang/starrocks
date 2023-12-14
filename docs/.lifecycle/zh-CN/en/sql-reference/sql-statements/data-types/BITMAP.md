---
displayed_sidebar: "Chinese"
---

# 位图

位图通常用于加速计数不同。在计数不同方面，它与HyperLogLog（HLL）相似但更准确。位图消耗的内存和磁盘资源更多。它仅支持对INT数据的聚合。如果要将位图应用于字符串数据，必须使用低基数字典映射数据。

本主题提供了一个简单的示例，说明如何创建位图列并使用位图函数对该列的数据进行聚合。有关详细的功能定义或更多位图函数，请参见"位图函数"。

## 创建表

- 创建一个聚合表，其中`user_id`列的数据类型为位图，并使用bitmap_union()函数对数据进行聚合。

    ```SQL
    CREATE TABLE `pv_bitmap` (
    `dt` int(11) NULL COMMENT "",
    `page` varchar(10) NULL COMMENT "",
    `user_id` bitmap BITMAP_UNION NULL COMMENT ""
    ) ENGINE=OLAP
    AGGREGATE KEY(`dt`, `page`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`dt`);
    ```

- 创建一个主键表，其中`userid`列的数据类型为位图。

    ```SQL
    CREATE TABLE primary_bitmap (
    `tagname` varchar(65533) NOT NULL COMMENT "标签名",
    `tagvalue` varchar(65533) NOT NULL COMMENT "标签值",
    `userid` bitmap NOT NULL COMMENT "用户ID")
    ENGINE=OLAP
    PRIMARY KEY(`tagname`, `tagvalue`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`tagname`);
    ```

在将数据插入位图列之前，必须首先使用to_bitmap()函数转换数据。

有关如何使用位图的详细信息，例如，将位图数据加载到表中，请参见[位图](../../sql-functions/aggregate-functions/bitmap.md)。