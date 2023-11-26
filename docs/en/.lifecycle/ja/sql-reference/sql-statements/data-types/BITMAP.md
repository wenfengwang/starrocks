---
displayed_sidebar: "Japanese"
---

# ビットマップ

ビットマップは、一意のカウントを高速化するためによく使用されます。ビットマップは、HyperLogLog（HLL）よりもカウントの一意性が高く、メモリとディスクリソースをより多く消費します。ビットマップはINTデータの集計のみをサポートしています。文字列データにビットマップを適用する場合は、低基数辞書を使用してデータをマッピングする必要があります。

このトピックでは、ビットマップ列を作成し、その列のデータを集計するためにビットマップ関数を使用する簡単な例を提供します。詳細な関数の定義やその他のビットマップ関数については、「ビットマップ関数」を参照してください。

## テーブルの作成

- ビットマップを使用してデータを集計するために、`user_id`列のデータ型がBITMAPであり、bitmap_union()関数が使用される集計テーブルを作成します。

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

- ビットマップを使用してデータを集計するために、`userid`列のデータ型がBITMAPであるプライマリキーテーブルを作成します。

    ```SQL
    CREATE TABLE primary_bitmap (
    `tagname` varchar(65533) NOT NULL COMMENT "タグ名",
    `tagvalue` varchar(65533) NOT NULL COMMENT "タグ値",
    `userid` bitmap NOT NULL COMMENT "ユーザーID")
    ENGINE=OLAP
    PRIMARY KEY(`tagname`, `tagvalue`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`tagname`);
    ```

ビットマップ列にデータを挿入する前に、まずto_bitmap()関数を使用してデータを変換する必要があります。

ビットマップの使用方法の詳細については、例えばビットマップデータをテーブルにロードする方法などは、[bitmap](../../sql-functions/aggregate-functions/bitmap.md)を参照してください。
