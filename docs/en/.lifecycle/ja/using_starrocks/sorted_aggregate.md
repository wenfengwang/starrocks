---
displayed_sidebar: "Japanese"
---

# ソートされたストリーミング集計

データベースシステムにおける一般的な集計方法には、ハッシュ集計とソート集計があります。

StarRocksでは、v2.5以降、**ソートされたストリーミング集計**をサポートしています。

## 動作原理

集計ノード（AGG）は、GROUP BYと集計関数の処理を主に担当しています。

ソートされたストリーミング集計では、キーのシーケンスに従ってGROUP BYキーを比較し、ハッシュテーブルを作成する必要がありません。これにより、集計に消費されるメモリリソースを効果的に削減することができます。集計のカーディナリティが高いクエリでは、ソートされたストリーミング集計により集計パフォーマンスが向上し、メモリ使用量が減少します。

以下の変数を設定することで、ソートされたストリーミング集計を有効にすることができます：

```SQL
set enable_sort_aggregate=true;
```

## 制限事項

- GROUP BYのキーはシーケンスを持つ必要があります。たとえば、ソートキーが `k1, k2, k3` の場合、以下のようなGROUP BYが許可されます：
  - `GROUP BY k1` および `GROUP BY k1, k2`
  - `GROUP BY k1, k3` はソートキーのシーケンスに従っていません。したがって、このような句に対してはソートされたストリーミング集計は有効になりません。
- 選択したパーティションは単一のパーティションである必要があります（同じキーが異なるパーティションの異なるマシンに分散される可能性があるため）。
- GROUP BYキーは、テーブルを作成する際に指定したバケットキーと同じ分布を持つ必要があります。たとえば、テーブルに `k1, k2, k3` の3つの列がある場合、バケットキーは `k1` または `k1, k2` になります。
  - バケットキーが `k1` の場合、`GROUP BY` キーは `k1`、`k1, k2`、または `k1, k2, k3` になります。
  - バケットキーが `k1, k2` の場合、`GROUP BY` キーは `k1, k2` または `k1, k2, k3` になります。
  - クエリプランがこの要件を満たさない場合、ソートされたストリーミング集計機能は有効になっていても効果がありません。
- ソートされたストリーミング集計は、最初のステージの集計にのみ適用されます（つまり、AGGノードの下にはScanノードが1つしかない場合）。

## 例

1. テーブルを作成し、データを挿入します。

    ```SQL
    CREATE TABLE `test_sorted_streaming_agg_basic`
    (
        `id_int` int(11) NOT NULL COMMENT "",
        `id_string` varchar(100) NOT NULL COMMENT ""
    ) 
    ENGINE=OLAP 
    DUPLICATE KEY(`id_int`)COMMENT "OLAP"
    DISTRIBUTED BY HASH(`id_int`)
    PROPERTIES
    ("replication_num" = "3"); 

    INSERT INTO test_sorted_streaming_agg_basic VALUES
    (1, 'v1'),
    (2, 'v2'),
    (3, 'v3'),
    (1, 'v4');
    ```

2. ソートされたストリーミング集計を有効にし、EXPLAINを使用してSQLプロファイルをクエリします。

    ```SQL
    set enable_sort_aggregate = true;

    explain costs select id_int, max(id_string)
    from test_sorted_streaming_agg_basic
    group by id_int;
    ```

## ソートされたストリーミング集計が有効かどうかを確認する

`EXPLAIN costs` の結果を表示します。AGGノードの `sorted streaming` フィールドが `true` の場合、この機能が有効になっています。

```Plain
|                                                                                                                                    |
|   1:AGGREGATE (update finalize)                                                                                                    |
|   |  aggregate: max[([2: id_string, VARCHAR, false]); args: VARCHAR; result: VARCHAR; args nullable: false; result nullable: true] |
|   |  group by: [1: id_int, INT, false]                                                                                             |
|   |  sorted streaming: true                                                                                                        |
|   |  cardinality: 1                                                                                                                |
|   |  column statistics:                                                                                                            |
|   |  * id_int-->[-Infinity, Infinity, 0.0, 1.0, 1.0] UNKNOWN                                                                       |
|   |  * max-->[-Infinity, Infinity, 0.0, 1.0, 1.0] UNKNOWN                                                                          |
|   |                                                                                                                                |
|   0:OlapScanNode                                                                                                                   |
|      table: test_sorted_streaming_agg_basic, rollup: test_sorted_streaming_agg_basic                                               |
|      preAggregation: on                                                                                                            |
|      partitionsRatio=1/1, tabletsRatio=10/10                                                                                       |
|      tabletList=30672,30674,30676,30678,30680,30682,30684,30686,30688,30690                                                        |
|      actualRows=0, avgRowSize=2.0                                                                                                  |
|      cardinality: 1                                                                                                                |
|      column statistics:                                                                                                            |
|      * id_int-->[-Infinity, Infinity, 0.0, 1.0, 1.0] UNKNOWN                                                                       |
|      * id_string-->[-Infinity, Infinity, 0.0, 1.0, 1.0] UNKNOWN                                                                    |
```
