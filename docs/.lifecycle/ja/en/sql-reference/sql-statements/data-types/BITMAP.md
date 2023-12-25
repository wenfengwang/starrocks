---
displayed_sidebar: English
---

# BITMAP

BITMAPは、カウントディスティンクトを高速化するためによく使用されます。HyperLogLog (HLL) と似ていますが、カウントディスティンクトにおいてはより正確です。BITMAPは、より多くのメモリとディスクリソースを消費します。INTデータの集約のみをサポートしています。文字列データにBITMAPを適用したい場合は、低カーディナリティ辞書を使用してデータをマップする必要があります。

このトピックでは、BITMAP列を作成し、BITMAP関数を使用してその列のデータを集約する方法の簡単な例を提供します。詳細な関数定義やその他のBitmap関数については、「Bitmap関数」を参照してください。

## テーブルの作成

- `user_id`列のデータ型がBITMAPで、bitmap_union()関数を使用してデータを集約する集約型テーブルを作成します。

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

- `userid`列のデータ型がBITMAPであるプライマリーキーテーブルを作成します。

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

BITMAP列にデータを挿入する前に、to_bitmap()関数を使用してデータを変換する必要があります。

BITMAPを使用する方法の詳細、例えばテーブルにBITMAPデータをロードする方法については、[bitmap](../../sql-functions/aggregate-functions/bitmap.md)を参照してください。
