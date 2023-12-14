---
displayed_sidebar: "Chinese"
---

# base64_to_bitmap

## 描述

在将位图数据导入StarRocks之前，您需要将数据序列化并将数据编码为Base64字符串。当您将Base64字符串导入StarRocks时，您需要将字符串转换为位图数据。
此功能用于将Base64字符串转换为位图数据。

此功能在v2.3中受支持。

## 语法

```Haskell
BITMAP base64_to_bitmap(VARCHAR bitmap)
```

## 参数

`bitmap`: 支持的数据类型为VARCHAR。在将位图数据加载到StarRocks之前，您可以使用Java或C ++ [创建BitmapValue对象](https://github.com/StarRocks/starrocks/blob/main/fe/spark-dpp/src/test/java/com/starrocks/load/loadv2/dpp/BitmapValueTest.java)，添加元素，序列化数据，并将数据编码为Base64字符串。然后，将Base64字符串作为输入参数传递给此函数。

## 返回值

返回BITMAP类型的值。

## 示例

创建名为`bitmapdb`的数据库和名为`bitmap`的表。使用Stream Load将JSON数据导入`bitmap_table`。在此过程中，使用`base64_to_bitmap`将JSON文件中的Base64字符串转换为位图数据。

1. 在StarRocks中创建数据库和表。在此示例中，创建了一个主键表。

    ```SQL
    CREATE database bitmapdb;
    USE bitmapdb;
    CREATE TABLE `bitmap_table` (
    `tagname` varchar(65533) NOT NULL COMMENT "标签名称",
    `tagvalue` varchar(65533) NOT NULL COMMENT "标签值",
    `userid` bitmap NOT NULL COMMENT "用户ID"
    ) ENGINE=OLAP
    PRIMARY KEY(`tagname`, `tagvalue`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`tagname`)
    PROPERTIES (
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
    );
    ```

2. 使用[Stream Load](../../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)将JSON数据导入`bitmap_table`。

    假设有一个名为**simpledata**的JSON文件。此文件具有以下内容，`userid`是Base64编码的字符串。

    ```JSON
    {
        "tagname": "产品", "tagvalue": "保险", "userid":"AjowAAABAAAAAAACABAAAAABAAIAAwA="
    }
    ```

    使用`base64_to_bitmap`将`userid`转换为位图值。

    ```Plain
    curl --location-trusted -u <username>:<password>\
        -H "columns: c1,c2,c3,tagname=c1,tagvalue=c2,userid=base64_to_bitmap(c3)"\
        -H "label:bitmap123"\
        -H "format: json"\
        -H "jsonpaths: [\"$.tagname\",\"$.tagvalue\",\"$.userid\"]"\
        -T simpleData http://host:port/api/bitmapdb/bitmap_table/_stream_load
    ```

3. 从`bitmap_table`中查询数据。

    ```Plaintext
    mysql> select tagname,tagvalue,bitmap_to_string(userid) from bitmap_table;
    +--------------+----------+----------------------------+
    | tagname      | tagvalue | bitmap_to_string(`userid`) |
    +--------------+----------+----------------------------+
    | 产品       | 保险       | 1,2,3                |
    +--------------+----------+----------------------------+
    1 行在集合中 (0.01 秒)
    ```