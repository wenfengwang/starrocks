---
displayed_sidebar: "Japanese"
---

# HLL_CARDINALITY

## 説明

HLL_CARDINALITYは、単一のHLLタイプの値の基数を計算するために使用されます。

## 構文

```Haskell
HLL_CARDINALITY(hll)
```

## 例

```plain text
MySQL > select HLL_CARDINALITY(uv_set) from test_uv;
+---------------------------+
| hll_cardinality(`uv_set`) |
+---------------------------+
|                         3 |
+---------------------------+
```

## キーワード

HLL,HLL_CARDINALITY
