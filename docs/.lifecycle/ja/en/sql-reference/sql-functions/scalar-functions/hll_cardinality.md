---
displayed_sidebar: English
---

# hll_cardinality

## 説明

単一のHLL型の値のカーディナリティを計算します。

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

HLL, HLL_CARDINALITY
