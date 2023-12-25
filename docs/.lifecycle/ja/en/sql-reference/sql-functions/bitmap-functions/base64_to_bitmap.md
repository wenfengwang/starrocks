---
displayed_sidebar: English
---

# base64_to_bitmap

## 説明

StarRocksにビットマップデータをインポートする前に、データをシリアライズし、Base64文字列としてエンコードする必要があります。StarRocksにBase64文字列をインポートする際には、その文字列をビットマップデータに変換する必要があります。
この関数はBase64文字列をビットマップデータに変換するために使用されます。

この関数はv2.3からサポートされています。

## 構文

```Haskell
BITMAP base64_to_bitmap(VARCHAR bitmap)
```

## パラメータ

`bitmap`: サポートされるデータ型はVARCHARです。StarRocksにBitmapデータをロードする前に、JavaまたはC++を使用して[BitmapValueオブジェクトを作成](https://github.com/StarRocks/starrocks/blob/main/fe/plugin-common/src/test/java/com/starrocks/types/BitmapValueTest.java)し、要素を追加してデータをシリアライズし、Base64文字列としてエンコードします。その後、この関数にBase64文字列を入力パラメータとして渡します。

## 戻り値

BITMAP型の値を返します。

## 例

`bitmapdb`というデータベースと`bitmap_table`というテーブルを作成します。このプロセス中に、base64_to_bitmapを使用してJSONファイル内のBase64文字列をビットマップデータに変換します。

1. StarRocksでデータベースとテーブルを作成します。この例では、プライマリキーテーブルが作成されます。

    ```SQL
    CREATE DATABASE bitmapdb;
    USE bitmapdb;
    CREATE TABLE `bitmap_table` (
    `tagname` VARCHAR(65533) NOT NULL COMMENT "タグ名",
    `tagvalue` VARCHAR(65533) NOT NULL COMMENT "タグ値",
    `userid` BITMAP NOT NULL COMMENT "ユーザーID"
    ) ENGINE=OLAP
    PRIMARY KEY(`tagname`, `tagvalue`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`tagname`)
    PROPERTIES (
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
    );
    ```

2. [Stream Load](../../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用して、`bitmap_table`にJSONデータをインポートします。

    **simpledata**という名前のJSONファイルがあるとします。このファイルには次の内容が含まれており、`userid`はBase64エンコードされた文字列です。

    ```JSON
    {
        "tagname": "Product", "tagvalue": "Insurance", "userid":"AjowAAABAAAAAAACABAAAAABAAIAAwA="
    }
    ```

    base64_to_bitmapを使用して`userid`をビットマップ値に変換します。

    ```Plain
    curl --location-trusted -u <username>:<password>\
        -H "columns: c1,c2,c3,tagname=c1,tagvalue=c2,userid=base64_to_bitmap(c3)"\
        -H "label:bitmap123"\
        -H "format: json"\
        -H "jsonpaths: [\"$.tagname\",\"$.tagvalue\",\"$.userid\"]"\
        -T simpleData http://host:port/api/bitmapdb/bitmap_table/_stream_load
    ```

3. `bitmap_table`からデータをクエリします。

    ```Plaintext
    mysql> SELECT tagname, tagvalue, bitmap_to_string(userid) FROM bitmap_table;
    +--------------+----------+----------------------------+
    | tagname      | tagvalue | bitmap_to_string(`userid`) |
    +--------------+----------+----------------------------+
    | Product      | Insurance| 1,2,3                      |
    +--------------+----------+----------------------------+
    1 rows in set (0.01 sec)
    ```
