---
displayed_sidebar: English
---

# BITMAP

BITMAP 通常用于加速去重计数。它与 HyperLogLog (HLL) 类似，但在去重计数上更为精确。BITMAP 消耗更多的内存和磁盘资源。它仅支持对 INT 数据类型的聚合。如果你想对字符串数据应用 BITMAP，必须使用低基数字典进行映射。

本主题提供了一个简单的示例，介绍如何创建一个 BITMAP 列，并使用 bitmap 函数对该列的数据进行聚合。有关函数的详细定义或更多 Bitmap 函数，请参见“Bitmap 函数”。

## 创建表

- 创建一个聚合表，其中 `user_id` 列的数据类型为 BITMAP，并使用 bitmap_union() 函数进行数据聚合。

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

- 创建一个主键表，其中 `userid` 列的数据类型为 BITMAP。

  ```SQL
  CREATE TABLE primary_bitmap (
  `tagname` varchar(65533) NOT NULL COMMENT "标签名称",
  `tagvalue` varchar(65533) NOT NULL COMMENT "标签值",
  `userid` bitmap NOT NULL COMMENT "用户 ID")
  ENGINE=OLAP
  PRIMARY KEY(`tagname`, `tagvalue`)
  COMMENT "OLAP"
  DISTRIBUTED BY HASH(`tagname`);
  ```

在将数据插入 BITMAP 列之前，必须首先使用 to_bitmap() 函数进行转换。

有关如何使用 BITMAP 的详细信息，例如如何将 BITMAP 数据加载到表中，请参阅 [bitmap](../../sql-functions/aggregate-functions/bitmap.md)。