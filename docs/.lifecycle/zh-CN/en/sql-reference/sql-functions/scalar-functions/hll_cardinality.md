---
displayed_sidebar: "Chinese"
---

# HLL_CARDINALITY

## 描述

HLL_CARDINALITY 用于计算单个 HLL 类型值的基数。

## 语法

```Haskell
HLL_CARDINALITY(hll)
```

## 示例

```plain text
MySQL > select HLL_CARDINALITY(uv_set) from test_uv;
+---------------------------+
| hll_cardinality(`uv_set`) |
+---------------------------+
|                         3 |
+---------------------------+
```

## 关键词

HLL, HLL_CARDINALITY