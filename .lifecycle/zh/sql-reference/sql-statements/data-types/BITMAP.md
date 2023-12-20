---
displayed_sidebar: English
---

# 位图

位图（BITMAP）通常用于加速唯一计数操作。它与 HyperLogLog（HLL）在功能上相似，但在唯一计数的准确性上更胜一筹。不过，位图消耗的内存和磁盘资源较多。它只支持对 INT 类型数据进行聚合操作。如果需要对字符串数据应用位图，你必须先通过低基数词典将数据映射转换。

本主题将通过一个简单的例子，介绍如何创建一个位图列以及如何使用位图函数对该列数据进行聚合。想要获取更多位图函数的详细定义，请参考“位图函数”部分。

## 创建表格

- 创建一个聚合表，在该表中，user_id 列的数据类型定义为 BITMAP，并使用 bitmap_union() 函数来聚合数据。

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

- 创建一个主键表，在该表中，userid 列的数据类型定义为 BITMAP。

  ```SQL
  CREATE TABLE primary_bitmap (
  `tagname` varchar(65533) NOT NULL COMMENT "Tag name",
  `tagvalue` varchar(65533) NOT NULL COMMENT "Tag value",
  `userid` bitmap NOT NULL COMMENT "User ID")
  ENGINE=OLAP
  PRIMARY KEY(`tagname`, `tagvalue`)
  COMMENT "OLAP"
  DISTRIBUTED BY HASH(`tagname`);
  ```

在向 BITMAP 列插入数据之前，必须先使用 to_bitmap() 函数对数据进行转换。

要了解更多关于如何使用 **BITMAP** 的信息，例如如何将 **BITMAP** 数据加载进表格，请参考[bitmap](../../sql-functions/aggregate-functions/bitmap.md)部分。
