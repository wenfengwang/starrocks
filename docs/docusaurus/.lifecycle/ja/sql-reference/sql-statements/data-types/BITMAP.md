---
displayed_sidebar: "Japanese"
---

# BITMAP（ビットマップ）

BITMAPは、頻繁なカウントの高速化によく使用されます。HLL（HyperLogLog）よりも、カウントの高速化においてより正確です。BITMAPは、HLLよりも多くのメモリとディスクリソースを消費します。INTデータの集約のみをサポートしています。文字列データにBITMAPを適用する場合は、低基数辞書を使用してデータをマップする必要があります。

このトピックでは、BITMAP列の作成とその列のデータを集約するためのビットマップ関数の使用例を示します。詳細な関数定義や他のビットマップ関数については、「ビットマップ関数」を参照してください。

## テーブルの作成

- ビットマップ(BITMAP)データ型の`user_id`列を持ち、bitmap_union()関数がデータを集約するために使用される、集約テーブルを作成します。

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

- ビットマップ(BITMAP)データ型の`userid`列を持つ、Primaru Keyテーブルを作成します。

    ```SQL
    CREATE TABLE primary_bitmap (
    `tagname` varchar(65533) NOT NULL COMMENT "Tag name",
    `tagvalue` varchar(65533) NOT NULL COMMENT "Tag value",
    `userid` bitmap NOT NULL COMMENT "User ID")
    ENGINE=OLAP
    PRIMARY KEY(`tagname`, `tagvalue`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`tagname`);
    ```

BITMAP列にデータを挿入する前に、まずto_bitmap()関数でデータを変換する必要があります。

BITMAPの使用方法の詳細については、BITMAPデータをテーブルにロードする方法などを参照してください[bitmap](../../sql-functions/aggregate-functions/bitmap.md)。