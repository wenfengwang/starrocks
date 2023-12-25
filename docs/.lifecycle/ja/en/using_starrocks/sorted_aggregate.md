---
displayed_sidebar: English
---

# ソート済みストリーミング集約

データベースシステムにおける一般的な集約方法には、ハッシュ集約とソート集約があります。

v2.5以降、StarRocksは**ソート済みストリーミング集約**をサポートしています。

## 動作原理

集約ノード（AGG）は、主にGROUP BYと集約関数の処理を担当します。

ソート済みストリーミング集約は、ハッシュテーブルを作成せずに、キーの順序に従ってGROUP BYキーを比較することでデータをグループ化できます。これにより、集約によって消費されるメモリリソースが効果的に削減されます。集約のカーディナリティが高いクエリでは、ソート済みストリーミング集約は集約パフォーマンスを向上させ、メモリ使用量を削減します。

ソート済みストリーミング集約を有効にするには、以下の変数を設定します：

```SQL
set enable_sort_aggregate=true;
```

## 制限事項

- GROUP BYのキーには順序が必要です。例えば、ソートキーが `k1, k2, k3` の場合：
  - `GROUP BY k1` と `GROUP BY k1, k2` は許可されます。
  - `GROUP BY k1, k3` はソートキーの順序に従っていません。したがって、ソート済みストリーミング集約はこのような句には適用できません。
- 選択されたパーティションは単一のパーティションでなければなりません（同じキーが異なるパーティションで異なるマシンに分散されている可能性があるため）。
- GROUP BYキーは、テーブル作成時に指定されたバケットキーと同じ分布を持つ必要があります。例えば、テーブルに `k1, k2, k3` の3つの列がある場合、バケットキーは `k1` または `k1, k2` です。
  - バケットキーが `k1` の場合、GROUP BYキーは `k1`、`k1, k2`、または `k1, k2, k3` にすることができます。
  - バケットキーが `k1, k2` の場合、GROUP BYキーは `k1, k2` または `k1, k2, k3` にすることができます。
  - クエリプランがこの要件を満たしていない場合、ソート済みストリーミング集約機能は、この機能が有効になっていても適用されません。
- ソート済みストリーミング集約は、第一段階の集約（つまり、AGGノードの下にScanノードが1つだけある場合）でのみ機能します。

## 例

1. テーブルを作成し、データを挿入します。

    ```SQL
    CREATE TABLE `test_sorted_streaming_agg_basic`
    (
        `id_int` int(11) NOT NULL COMMENT "",
        `id_string` varchar(100) NOT NULL COMMENT ""
    ) 
    ENGINE=OLAP 
    DUPLICATE KEY(`id_int`) COMMENT "OLAP"
    DISTRIBUTED BY HASH(`id_int`)
    PROPERTIES
    ("replication_num" = "3"); 

    INSERT INTO test_sorted_streaming_agg_basic VALUES
    (1, 'v1'),
    (2, 'v2'),
    (3, 'v3'),
    (1, 'v4');
    ```

2. ソート済みストリーミング集約を有効にし、EXPLAINを使用してSQLプロファイルを確認します。

    ```SQL
    set enable_sort_aggregate = true;

    explain costs select id_int, max(id_string)
    from test_sorted_streaming_agg_basic
    group by id_int;
    ```

## ソート済みストリーミング集約が有効かどうかを確認する

`EXPLAIN costs` の結果を確認します。AGGノードに `sorted streaming` フィールドが `true` と表示されている場合、この機能は有効です。

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
