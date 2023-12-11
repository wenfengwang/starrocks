---
displayed_sidebar: "Japanese"
---

# ソートされたストリーミング集約

データベースシステムにおける一般的な集約方法には、ハッシュ集約とソート集約があります。

v2.5以降、StarRocksは**ソートされたストリーミング集約**をサポートしています。

## 動作原理

集約ノード（AGG）は主にGROUP BYおよび集約関数の処理を担当しています。

ソートされたストリーミング集約は、キーのシーケンスに従ってGROUP BYキーを比較することでデータをグループ化し、ハッシュテーブルを作成する必要はありません。これにより、集約に消費されるメモリリソースを効果的に削減できます。集約の基数が高いクエリに対しては、ソートされたストリーミング集約が集約パフォーマンスを向上させ、メモリの使用量も削減します。

次の変数を設定することで、ソートされたストリーミング集約を有効にできます。

```SQL
set enable_sort_aggregate=true;
```

## 制限事項

- GROUP BY句のキーはシーケンスを持たなければなりません。たとえば、ソートキーが `k1, k2, k3` の場合、次のようになります：
  - `GROUP BY k1` および  `GROUP BY k1, k2` が許可されます。
  - `GROUP BY k1, k3` はソートキーのシーケンスに従っていません。したがって、ソートされたストリーミング集約はそのような句に対して有効になりません。
- 選択したパーティションは単一のパーティションである必要があります（同じキーが異なるパーティションの異なるマシンに分散される可能性があるため）。
- GROUP BYキーは、テーブル作成時に指定されたバケットキーと同じ分布を持たなければなりません。たとえば、テーブルに `k1, k2, k3` の3つの列がある場合、バケットキーは `k1` または `k1, k2` になります。
  - バケットキーが `k1` の場合、`GROUP BY`キーは `k1`、`k1, k2`、または `k1, k2, k3` になります。
  - バケットキーが `k1, k2` の場合、`GROUP BY`キーは `k1, k2` または `k1, k2, k3` になります。
  - クエリプランがこの要件を満たさない場合、ソートされたストリーミング集約機能は有効になっていても効果がありません。
- ソートされたストリーミング集約は、最初の段階の集約のみに適用されます（つまり、AGGノードの下にはScanノードが1つしかない場合）。

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

2. ソートされたストリーミング集約を有効にし、EXPLAINコマンドを使用してSQLプロファイルをクエリします。

    ```SQL
    set enable_sort_aggregate = true;

    explain costs select id_int, max(id_string)
    from test_sorted_streaming_agg_basic
    group by id_int;
    ```

## ソートされたストリーミング集約が有効かどうかを確認する

`EXPLAIN costs`の結果を表示します。AGGノードの`sorted streaming`が`true`であれば、この機能が有効になっています。

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