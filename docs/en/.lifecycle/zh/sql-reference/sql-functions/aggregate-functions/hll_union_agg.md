---
displayed_sidebar: English
---

# hll_union_agg

## 描述

HLL是基于HyperLogLog算法的工程实现，用于保存HyperLogLog计算过程的中间结果。

它只能作为表的值列，通过聚合减少数据量，以加快查询的速度。

基于HLL的估计结果，误差约为1%。HLL列是由其他列生成的，或者基于加载到表中的数据生成的。

在加载过程中，使用[hll_hash](../aggregate-functions/hll_hash.md)函数来指定用于生成HLL列的列。它通常用于替换Count Distinct，并通过组合rollup在业务中快速计算UV。

## 语法

```Haskell
HLL_UNION_AGG(hll)
```

## 例子

```plain text
MySQL > select HLL_UNION_AGG(uv_set) from test_uv;
+-------------------------+
| HLL_UNION_AGG(`uv_set`) |
+-------------------------+
| 17721                   |
+-------------------------+
```

## 关键词

HLL_UNION_AGG，HLL，UNION，AGG