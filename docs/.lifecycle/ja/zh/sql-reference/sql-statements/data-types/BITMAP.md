---
displayed_sidebar: Chinese
---

# BITMAP

## 説明

BITMAPはHLL（HyperLogLog）に似ており、count distinctの高速な重複カウントによく使用されます。

bitmap関数を使用してさまざまな集合操作を行うことができ、HLLよりも正確な結果を得ることができます。ただし、BITMAPはより多くのメモリとディスクリソースを消費し、整数型のみの集約をサポートしています。文字列などの他のタイプの場合は、辞書を使用してマッピングする必要があります。

## 例

1. 聚合モデルでテーブルを作成する際に、フィールドタイプとしてBITMAPを指定します。

    ```sql
    CREATE TABLE pv_bitmap (
        dt INT(11) NULL COMMENT "",
        page VARCHAR(10) NULL COMMENT "",
        user_id bitmap BITMAP_UNION NULL COMMENT ""
    ) ENGINE=OLAP
    AGGREGATE KEY(dt, page)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(dt);
    ```

2. 主キーモデルでテーブルを作成する際に、フィールドタイプとしてBITMAPを指定します。

    ```sql
    CREATE TABLE primary_bitmap (
      `tagname` varchar(65533) NOT NULL COMMENT "タグ名",
      `tagvalue` varchar(65533) NOT NULL COMMENT "タグ値",
      `userid` bitmap NOT NULL COMMENT "ユーザーID"
    ) ENGINE=OLAP
    PRIMARY KEY(`tagname`, `tagvalue`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`tagname`);
    ```

BITMAP列にデータを挿入するには、先にto_bitmap()関数を使用して変換する必要があります。

BITMAPタイプの詳細な使用方法、例えばテーブルにBITMAP値を挿入する方法については、[bitmap](../../sql-functions/aggregate-functions/bitmap.md)を参照してください。

BITMAPタイプのフィールドは、bitmap_and()、bitmap_andnot()など、多くのBITMAP関数をサポートしています。詳細は[bitmap-functions](../../sql-functions/bitmap-functions/bitmap_and.md)を参照してください。
