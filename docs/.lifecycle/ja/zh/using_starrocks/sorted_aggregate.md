---
displayed_sidebar: Chinese
---

# ソート済みストリーミング集計

データベースで一般的な集計方法には、Hash集計とソート集計があります。

バージョン2.5から、StarRocksはソート済みストリーミング集計（Sorted streaming aggregate）をサポートしています。

## 原理

集計ノードは主にGROUP BYによる分類と集計関数の計算を処理します。

Sorted streaming aggregateは、入力キーの順序性に基づいて、GROUP BY列を直接比較する方法で直接グループ化を行い、ハッシュテーブルを構築する必要がないため、集計計算のメモリ使用量を効果的に削減できます。集計基数が高い場合、集計性能を向上させ、メモリ使用量を削減することができます。

Sorted streaming aggregateを有効にするには、次の変数を設定します:

```SQL
set enable_sort_aggregate=true;
```

## 使用上の制限

- GROUP BYに含まれるキーはソート済みである必要があります:
  例えば、テーブルのソート列が`k1,k2,k3`の場合:
  - `GROUP BY k1`や`GROUP BY k1, k2`は可能です。
  - `GROUP BY k1, k3`はソートを保証できないため、Sorted streaming aggregateは有効になりません。
- 選択されたパーティションは1つだけでなければなりません（異なるパーティションに同じキーが異なるマシンに分散している可能性があるため）。
- GROUP BYに含まれるキーは、テーブル作成時に指定されたバケットキーの分布順序と一致している必要があります。
  例えば、`k1,k2,k3`の3つの列がある場合、`k1`または`k1,k2`に基づいてデータをバケット分けすることができます。
  - `k1`に基づいてバケット分けする場合、`GROUP BY k1`、`GROUP BY k1, k2`、`GROUP BY k1,k2,k3`がサポートされます。
  - `k1,k2`に基づいてバケット分けする場合、`GROUP BY k1,k2`、`GROUP BY k1,k2,k3`がサポートされます。
  - 条件に合わないクエリプランについては、`enable_sort_aggregate`を有効にしても、sorted streaming aggregateは有効になりません。
- 1段階集計でなければなりません（つまり、AGGノードの下にはScanノードが1つだけ接続されている）。

## 使用例

1. テーブルを作成し、データを挿入します。

    ```SQL
    CREATE TABLE `test_sorted_streaming_agg_basic`
    (
        `id_int` int(11) NOT NULL COMMENT "",
        `id_string` varchar(100) NOT NULL COMMENT ""
    )
    ENGINE=OLAP 
    DUPLICATE KEY(`id_int`)
    COMMENT "OLAP"
    DISTRIBUTED BY HASH(`id_int`)
    PROPERTIES (
    "replication_num" = "3"
    ); 

    INSERT INTO test_sorted_streaming_agg_basic VALUES
    (1, 'v1'),
    (2, 'v2'),
    (3, 'v3'),
    (1, 'v4');
    ```

2. Sorted streaming aggregateを有効にします。EXPLAINで確認します。

    ```SQL
    set enable_sort_aggregate = true;

    explain costs select id_int, max(id_string)
    from test_sorted_streaming_agg_basic
    group by id_int;
    ```

## 有効化の確認

Explain costsの結果を確認します。AGGノード内にsorted streamingフィールドが見つかれば、有効になっていることが証明されます。

```C++
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
