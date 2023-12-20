---
displayed_sidebar: English
---

# hll_cardinality

## 描述

计算单个 HLL 类型值的基数。

## 语法

```Haskell
HLL_CARDINALITY(hll)
```

## 示例

```plain
MySQL > select HLL_CARDINALITY(uv_set) from test_uv;
+---------------------------+
| hll_cardinality(`uv_set`) |
+---------------------------+
|                         3 |
+---------------------------+
```

## 关键词

HLL, HLL_CARDINALITY