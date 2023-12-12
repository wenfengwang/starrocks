---
displayed_sidebar: "Japanese"
---

# base64_to_bitmap

## 説明

StarRocksにビットマップデータをインポートする前に、データをシリアライズし、Base64文字列としてエンコードする必要があります。Base64文字列をStarRocksにインポートする際、文字列をビットマップデータに変換する必要があります。
この関数は、Base64文字列をビットマップデータに変換するために使用されます。

この関数はv2.3からサポートされています。

## 構文

```Haskell
BITMAP base64_to_bitmap(VARCHAR bitmap)
```

## パラメータ

`bitmap`: サポートされているデータ型はVARCHARです。BitmapデータをStarRocksにロードする前に、JavaまたはC ++を使用して[BitmapValueオブジェクトを作成](https://github.com/StarRocks/starrocks/blob/main/fe/spark-dpp/src/test/java/com/starrocks/load/loadv2/dpp/BitmapValueTest.java)し、要素を追加し、データをシリアライズし、データをBase64文字列としてエンコードすることができます。その後、この関数に入力パラメータとしてBase64文字列を渡します。

## 戻り値

BITMAPタイプの値を返します。

## 例

データベース`bitmapdb`とテーブル`bitmap`を作成します。JSONデータを`bitmap_table`にインポートする際、このプロセス中にbase64_to_bitmapを使用してJSONファイル内のBase64文字列をビットマップデータに変換します。

1. StarRocksでデータベースとテーブルを作成します。この例では、プライマリキーテーブルが作成されます。

    ```SQL
    CREATE database bitmapdb;
    USE bitmapdb;
    CREATE TABLE `bitmap_table` (
    `tagname` varchar(65533) NOT NULL COMMENT "タグ名",
    `tagvalue` varchar(65533) NOT NULL COMMENT "タグ値",
    `userid` bitmap NOT NULL COMMENT "ユーザーID"
    ) ENGINE=OLAP
    PRIMARY KEY(`tagname`, `tagvalue`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`tagname`)
    PROPERTIES (
    "replication_num" = "3",
    "storage_format" = "DEFAULT"
    );
    ```

2. [Stream Load](../../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用してJSONデータを`bitmap_table`にインポートします。

    `simpledata`という名前のJSONファイルがあるとします。このファイルには以下の内容があり、`userid`はBase64でエンコードされた文字列です。

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
    mysql> select tagname,tagvalue,bitmap_to_string(userid) from bitmap_table;
    +--------------+----------+----------------------------+
    | tagname      | tagvalue | bitmap_to_string(`userid`) |
    +--------------+----------+----------------------------+
    | Product      | Insurance      | 1,2,3                |
    +--------------+----------+----------------------------+
    1 rows in set (0.01 sec)
    ```