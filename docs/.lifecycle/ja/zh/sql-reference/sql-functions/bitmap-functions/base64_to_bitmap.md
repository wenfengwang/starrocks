---
displayed_sidebar: Chinese
---

# base64_to_bitmap

## 機能

外部から bitmap データを StarRocks にインポートする際には、まず bitmap データをシリアライズして Base64 エンコードし、Base64 文字列を生成する必要があります。その後、StarRocks に文字列をインポートする際に、Base64 から bitmap に変換します。
この関数は、Base64 エンコードされた文字列を bitmap に変換するために使用されます。

この関数はバージョン 2.3 からサポートされています。

## 文法

```Haskell
BITMAP base64_to_bitmap(VARCHAR bitmap)
```

## パラメータ説明

`bitmap`：サポートされるデータタイプは VARCHAR です。外部から bitmap データをインポートする際には、Java または C++ のインターフェースを使用して最初に[BitmapValue オブジェクトを作成](https://github.com/StarRocks/starrocks/blob/main/fe/plugin-common/src/test/java/com/starrocks/types/BitmapValueTest.java)し、要素を追加してシリアライズし、Base64 エンコードを行った後、生成された Base64 文字列をこの関数の引数として使用します。

## 戻り値の説明

BITMAP 型のデータを返します。

## 例

`bitmapdb.bitmap_table` というデータベーステーブルを作成し、Stream Load を使用して JSON 形式のデータを `bitmap_table` にインポートします。このプロセスでは base64_to_bitmap 関数を使用してデータ変換を行います。

1. StarRocks でデータベースとテーブルを作成します。例として、PRIMARY KEY モデルのテーブルを作成する場合は以下のようにします。

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

2. [Stream Load](../../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) を使用して JSON 形式のデータを `bitmap_table` にインポートします。

    JSON 形式のファイル **simpledata** の内容は以下の通りで、`userid` は Base64 エンコードされた文字列です。

    ```JSON
    {
        "tagname": "持有产品",
        "tagvalue": "保险",
        "userid":"AjowAAABAAAAAAACABAAAAABAAIAAwA="
    }
    ```

    - JSON ファイルのデータを `bitmap_table` にインポートし、base64_to_bitmap 関数を使用して `userid` を bitmap に変換します。

    ```Plain Text
    curl --location-trusted -u <username>:<password>\
        -H "columns: c1,c2,c3,tagname=c1,tagvalue=c2,userid=base64_to_bitmap(c3)"\
        -H "label:bitmap123"\
        -H "format: json" -H "jsonpaths: [\"$.tagname\",\"$.tagvalue\",\"$.userid\"]"\
        -T simpleData http://<host:port>/api/bitmapdb/bitmap_table/_stream_load
    ```

3. `bitmap_table` テーブルのデータを照会します。

    ```Plain Text
    mysql> SELECT tagname,tagvalue,bitmap_to_string(userid) FROM bitmap_table;
    +--------------+----------+----------------------------+
    | tagname      | tagvalue | bitmap_to_string(`userid`) |
    +--------------+----------+----------------------------+
    | 持有产品      | 保险      | 1,2,3                      |
    +--------------+----------+----------------------------+
    1 rows in set (0.01 sec)
    ```
