---
displayed_sidebar: "Japanese"
---

# hll_union_agg（HLLの集約）

## Description（概要）

HLLは、HyperLogLogアルゴリズムに基づいたエンジニアリング実装であり、HyperLogLog計算プロセスの中間結果を保存するために使用されます。

これは、テーブルの値列としてのみ使用でき、集約を通じてデータ量を削減し、クエリの高速化を実現する目的で使用されます。

HLLに基づいた約1％の誤差を持つ推定結果。HLL列は、他の列によって生成されるか、テーブルにロードされたデータに基づいて生成されます。

ロード中、[hll_hash](../aggregate-functions/hll_hash.md) 関数が使用され、HLL列の生成に使用される列を指定します。これは、Count Distinctの代替としてよく使用され、ロールアップを組み合わせてビジネスで迅速にUVを計算するためによく使用されます。

## Syntax（構文）

```Haskell
HLL_UNION_AGG(hll)
```

## Examples（例）

```plain text
MySQL > select HLL_UNION_AGG(uv_set) from test_uv;
+-------------------------+
| HLL_UNION_AGG(`uv_set`) |
+-------------------------+
| 17721                   |
+-------------------------+
```

## keyword（キーワード）

HLL_UNION_AGG, HLL, UNION, AGG