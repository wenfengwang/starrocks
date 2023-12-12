---
displayed_sidebar: "Japanese"
---

# ビットマップ

ビットマップは、カウントの重複を加速するためによく使用されます。これはHyperLogLog（HLL）よりも正確なカウントの重複を持っていますが、より多くのメモリとディスクリソースを消費します。ビットマップはINTデータの集約のみをサポートしています。文字列データにビットマップを適用したい場合は、低基数辞書を使用してデータをマップする必要があります。

このトピックでは、ビットマップカラムを作成し、そのカラムのデータを集約するためのビットマップ関数を使用する簡単な例を提供します。詳細な関数の定義やその他のビットマップ関数については、「ビットマップ関数」を参照してください。

## テーブルの作成

- ビットマップ型の`user_id`カラムを持ち、bitmap_union()関数を使用してデータを集約する集約テーブルを作成します。

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

- プライマリキーとしてビットマップ型の`userid`カラムを持つテーブルを作成します。

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

ビットマップカラムにデータを挿入する前に、まずto_bitmap()関数を使用してデータを変換する必要があります。

ビットマップの使用方法の詳細については、たとえば、ビットマップデータをテーブルにロードする方法などについては、[bitmap](../../sql-functions/aggregate-functions/bitmap.md)を参照してください。