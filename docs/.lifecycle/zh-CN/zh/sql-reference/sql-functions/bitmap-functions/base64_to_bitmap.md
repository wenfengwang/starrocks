---
displayed_sidebar: "Chinese"
---

# base64_to_bitmap

## 功能

When importing external bitmap data into StarRocks, the bitmap data needs to be serialized and Base64 encoded to generate a Base64 string. When importing the string into StarRocks, the Base64 is then converted to bitmap.
This function is used to convert the Base64 encoded string to a bitmap.

This function is supported starting from version 2.3.

## 语法

```Haskell
BITMAP base64_to_bitmap(VARCHAR bitmap)
```

## 参数说明

`bitmap`: The supported data type is VARCHAR. When importing external bitmap data, you can use the Java or C++ interface to [create a BitmapValue object](https://github.com/StarRocks/starrocks/blob/main/fe/spark-dpp/src/test/java/com/starrocks/load/loadv2/dpp/BitmapValueTest.java), then add elements, serialize, Base64 encode, and use the resulting Base64 string as the input for this function.

## 返回值说明

Returns data of type BITMAP.

## 示例

Create the table `bitmapdb.bitmap_table` and use Stream Load to import JSON formatted data into the `bitmap_table`, using the base64_to_bitmap function for data conversion.

1. Create a database and table in StarRocks, taking the example of creating a table with a primary key model (PRIMARY KEY).

    ```SQL
    CREATE database bitmapdb;
    USE bitmapdb;
    CREATE TABLE `bitmap_table` (
    `tagname` varchar(65533) NOT NULL COMMENT "Tag Name",
    `tagvalue` varchar(65533) NOT NULL COMMENT "Tag Value",
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

2. Use [Stream Load](../../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) to import JSON formatted data into the `bitmap_table`.

    Assuming a JSON formatted file **simpledata** with contents as follows, where `userid` is the Base64 encoded string:

    ```JSON
    {
        "tagname": "Product Type",
        "tagvalue": "Insurance",
        "userid": "AjowAAABAAAAAAACABAAAAABAAIAAwA="
    }
    ```

    - Import the data from the JSON file into `bitmap_table`, using the base64_to_bitmap function to convert `userid` to a bitmap.

    ```Plain Text
    curl --location-trusted -u <username>:<password>\
        -H "columns: c1,c2,c3,tagname=c1,tagvalue=c2,userid=base64_to_bitmap(c3)"\
        -H "label:bitmap123"\
        -H "format: json" -H "jsonpaths: [\"$.tagname\",\"$.tagvalue\",\"$.userid\"]"\
        -T simpleData http://<host:port>/api/bitmapdb/bitmap_table/_stream_load
    ```

3. Query data from the `bitmap_table` table.

    ```Plain Text
    mysql> select tagname,tagvalue,bitmap_to_string(userid) from bitmap_table;
    +--------------+----------+----------------------------+
    | tagname      | tagvalue | bitmap_to_string(`userid`) |
    +--------------+----------+----------------------------+
    | Product Type | Insurance | 1,2,3                      |
    +--------------+----------+----------------------------+
    1 rows in set (0.01 sec)
    ```