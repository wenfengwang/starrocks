---
displayed_sidebar: "Japanese"
---

# HLL_UNION_AGG

## 説明

HLLは、HyperLogLogアルゴリズムに基づいたエンジニアリング実装であり、HyperLogLogの計算プロセスの中間結果を保存するために使用されます。

これは、テーブルの値列としてのみ使用でき、集計を通じてデータ量を削減し、クエリの高速化を実現する目的で使用されます。

HLLに基づいた誤差約1%の推定結果です。HLL列は、他の列によって生成されるか、テーブルにロードされたデータに基づいて生成されます。

ロード中に、[hll_hash](../aggregate-functions/hll_hash.md)関数を使用して、HLL列を生成するために使用する列を指定します。これは、Count Distinctの代わりとして使用され、ロールアップを組み合わせることでビジネスでのUVの高速な計算を行います。

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
