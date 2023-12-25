---
displayed_sidebar: English
---

# hll_union_agg

## 説明

HLLはHyperLogLogアルゴリズムに基づいたエンジニアリング実装で、HyperLogLog計算プロセスの中間結果を保存するために使用されます。

これはテーブルの値のカラムとしてのみ使用可能で、集約を通じてデータ量を減らし、クエリの速度を向上させる目的を達成します。

HLLに基づく約1%の誤差を持つ推定結果です。HLLカラムは他のカラムから生成されるか、テーブルにロードされたデータに基づいて生成されます。

ロード中には、[hll_hash](../aggregate-functions/hll_hash.md)関数を使用して、HLLカラムを生成するためのカラムを指定します。これはCount Distinctを置き換えたり、ロールアップと組み合わせてビジネスでUVを迅速に計算するためによく使用されます。

## 構文

```Haskell
HLL_UNION_AGG(hll)
```

## 例

```plain text
MySQL > select HLL_UNION_AGG(uv_set) from test_uv;
+-------------------------+
| HLL_UNION_AGG(`uv_set`) |
+-------------------------+
| 17721                   |
+-------------------------+
```

## キーワード

HLL_UNION_AGG, HLL, UNION, AGG
