---
displayed_sidebar: Chinese
---

# hll_union_agg

## 機能

この関数は複数の HLL 型データを一つの HLL に統合します。

HLL は HyperLogLog アルゴリズムに基づいた実装で、HyperLogLog の計算過程の中間結果を保持するために使用されます。

これはテーブルの value 列の型としてのみ使用でき、集約を通じてデータ量を減らし続けることで、クエリの速度を向上させる目的を実現します。

その結果は推定値であり、誤差はおよそ 1% 程度です。hll 列は他の列やインポートされたデータから生成されます。

データをインポートする際には、[hll_hash](../aggregate-functions/hll_hash.md) 関数を使用して、どの列を hll 列の生成に使用するかを指定します。これは count distinct の代わりとしてよく使用され、Rollup と組み合わせてビジネスで UV などの迅速な計算に利用されます。

## 文法

```Haskell
HLL_UNION_AGG(hll)
```

## パラメータ説明

`hll`: 他の列やインポートされたデータから生成された hll 列。

## 戻り値の説明

戻り値は数値型です。

## 例

```plain text
MySQL > select HLL_UNION_AGG(uv_set)
from test_uv;;
+-------------------------+
| HLL_UNION_AGG(`uv_set`) |
+-------------------------+
| 17721                   |
+-------------------------+
```
