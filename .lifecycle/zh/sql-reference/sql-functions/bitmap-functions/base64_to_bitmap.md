---
displayed_sidebar: English
---

# base64_to_bitmap

## 描述

在您将位图数据导入StarRocks之前，需要对数据进行序列化并将其编码为Base64字符串。当您将Base64字符串导入到StarRocks时，您需要将字符串转换成位图数据。此函数用于将Base64字符串转换成位图数据。

该函数从v2.3版本开始支持。

## 语法

```Haskell
BITMAP base64_to_bitmap(VARCHAR bitmap)
```

## 参数

`bitmap`：支持的数据类型是VARCHAR。在您将Bitmap数据导入到StarRocks之前，可以使用Java或C++来[创建一个BitmapValue对象](https://github.com/StarRocks/starrocks/blob/main/fe/plugin-common/src/test/java/com/starrocks/types/BitmapValueTest.java)，添加一个元素，序列化数据，并将数据编码为Base64字符串。然后，将Base64字符串作为输入参数传递给此函数。

## 返回值

返回BITMAP类型的值。

## 示例

创建一个名为bitmapdb的数据库和一个名为bitmap的表。使用Stream Load功能将JSON数据导入到bitmap_table中。在这个过程中，使用base64_to_bitmap函数将JSON文件中的Base64字符串转换成位图数据。

1. 在StarRocks中创建数据库和表。在这个例子中，创建了一个包含主键的表。

   ```SQL
   CREATE database bitmapdb;
   USE bitmapdb;
   CREATE TABLE `bitmap_table` (
   `tagname` varchar(65533) NOT NULL COMMENT "Tag name",
   `tagvalue` varchar(65533) NOT NULL COMMENT "Tag value",
   `userid` bitmap NOT NULL COMMENT "User ID"
   ) ENGINE=OLAP
   PRIMARY KEY(`tagname`, `tagvalue`)
   COMMENT "OLAP"
   DISTRIBUTED BY HASH(`tagname`)
   PROPERTIES (
   "replication_num" = "3",
   "storage_format" = "DEFAULT"
   );
   ```

2. 使用[Stream Load](../../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)功能将JSON数据导入到 `bitmap_table` 中。

   假设有一个名为**simpledata**.json的文件。该文件包含以下内容，其中`userid`是一个Base64编码的字符串。

   ```JSON
   {
       "tagname": "Product", "tagvalue": "Insurance", "userid":"AjowAAABAAAAAAACABAAAAABAAIAAwA="
   }
   ```

   使用base64_to_bitmap函数将userid转换为位图值。

   ```Plain
   curl --location-trusted -u <username>:<password>\
       -H "columns: c1,c2,c3,tagname=c1,tagvalue=c2,userid=base64_to_bitmap(c3)"\
       -H "label:bitmap123"\
       -H "format: json"\
       -H "jsonpaths: [\"$.tagname\",\"$.tagvalue\",\"$.userid\"]"\
       -T simpleData http://host:port/api/bitmapdb/bitmap_table/_stream_load
   ```

3. 从bitmap_table中查询数据。

   ```Plaintext
   mysql> select tagname,tagvalue,bitmap_to_string(userid) from bitmap_table;
   +--------------+----------+----------------------------+
   | tagname      | tagvalue | bitmap_to_string(`userid`) |
   +--------------+----------+----------------------------+
   | Product      | Insurance      | 1,2,3                |
   +--------------+----------+----------------------------+
   1 rows in set (0.01 sec)
   ```
