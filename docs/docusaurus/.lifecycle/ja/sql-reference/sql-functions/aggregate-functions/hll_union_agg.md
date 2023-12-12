---
displayed_sidebar: "Japanese"
---

# hll_union_agg

## 説明

HLLは、HyperLogLogアルゴリズムに基づいたエンジニアリング実装であり、HyperLogLog計算プロセスの中間結果を保存するために使用されます。

この関数は、テーブルの値列としてのみ使用でき、集約を通じてデータ量を減らし、クエリの高速化を実現します。

HLLに基づく誤差約1％の推定結果を出力します。HLL列は、他の列によって生成されるか、テーブルにロードされるデータに基づいて生成されます。

ロード中には、[hll_hash](../aggregate-functions/hll_hash.md) 関数が使用され、HLL列の生成に使用する列を指定します。これは、Count Distinctを置き換えたり、ロールアップを組み合わせてビジネスで迅速にUVを計算するためによく使用されます。

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

HLL_UNION_AGG,HLL,UNION,AGG