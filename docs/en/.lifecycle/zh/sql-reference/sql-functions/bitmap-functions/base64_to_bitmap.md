---
displayed_sidebar: English
---

# base64_to_bitmap

## 描述

在您将位图数据导入 StarRocks 之前，需要序列化数据并将数据编码为 Base64 字符串。当您将 Base64 字符串导入 StarRocks 时，需要将字符串转换为位图数据。此函数用于将 Base64 字符串转换为位图数据。

该函数从 v2.3 版本开始支持。

## 语法

```Haskell
BITMAP base64_to_bitmap(VARCHAR bitmap)
```

## 参数

`bitmap`: 支持的数据类型是 VARCHAR。在您将 Bitmap 数据加载到 StarRocks 之前，可以使用 Java 或 C++ [创建一个 BitmapValue 对象](https://github.com/StarRocks/starrocks/blob/main/fe/plugin-common/src/test/java/com/starrocks/types/BitmapValueTest.java)，添加元素，序列化数据，并将数据编码为 Base64 字符串。然后，将 Base64 字符串作为输入参数传递给此函数。

## 返回值

返回 BITMAP 类型的值。

## 示例

创建名为 `bitmapdb` 的数据库和名为 `bitmap` 的表。使用 Stream Load 将 JSON 数据导入 `bitmap_table`。在此过程中，使用 base64_to_bitmap 将 JSON 文件中的 Base64 字符串转换为位图数据。

1. 在 StarRocks 中创建数据库和表。在此示例中，创建了一个主键表。

   ```SQL
   CREATE DATABASE bitmapdb;
   USE bitmapdb;
   CREATE TABLE `bitmap_table` (
   `tagname` VARCHAR(65533) NOT NULL COMMENT "标签名",
   `tagvalue` VARCHAR(65533) NOT NULL COMMENT "标签值",
   `userid` BITMAP NOT NULL COMMENT "用户 ID"
   ) ENGINE=OLAP
   PRIMARY KEY(`tagname`, `tagvalue`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`tagname`)
   PROPERTIES (
   "replication_num" = "3",
   "storage_format" = "DEFAULT"
   );
   ```

2. 使用 [Stream Load](../../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 将 JSON 数据导入 `bitmap_table`。

   假设有一个名为 **simpledata** 的 JSON 文件。该文件包含以下内容，`userid` 是 Base64 编码的字符串。

   ```JSON
   {
       "tagname": "Product", "tagvalue": "Insurance", "userid":"AjowAAABAAAAAAACABAAAAABAAIAAwA="
   }
   ```

   使用 base64_to_bitmap 将 `userid` 转换为位图值。

   ```Plain
   curl --location-trusted -u <username>:<password>\
       -H "columns: c1,c2,c3,tagname=c1,tagvalue=c2,userid=base64_to_bitmap(c3)"\
       -H "label:bitmap123"\
       -H "format: json"\
       -H "jsonpaths: [\"$.tagname\",\"$.tagvalue\",\"$.userid\"]"\
       -T simpleData http://host:port/api/bitmapdb/bitmap_table/_stream_load
   ```

3. 从 `bitmap_table` 中查询数据。

   ```Plaintext
   mysql> SELECT tagname, tagvalue, bitmap_to_string(userid) FROM bitmap_table;
   +--------------+----------+----------------------------+
   | tagname      | tagvalue | bitmap_to_string(`userid`) |
   +--------------+----------+----------------------------+
   | Product      | Insurance | 1,2,3                     |
   +--------------+----------+----------------------------+
   1 row in set (0.01 sec)
   ```