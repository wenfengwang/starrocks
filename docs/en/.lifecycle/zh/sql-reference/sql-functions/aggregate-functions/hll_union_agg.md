---
displayed_sidebar: English
---

# hll_union_agg

## 描述

HLL 是基于 HyperLogLog 算法的工程实现，用于保存 HyperLogLog 计算过程的中间结果。

它只能作为表的值列使用，并通过聚合减少数据量，以此来加快查询速度。

基于 HLL 的估算结果误差大约为 1%。HLL 列是由其他列生成的，或者是基于加载到表中的数据生成的。

在加载数据时，使用 [hll_hash](../aggregate-functions/hll_hash.md) 函数来指定哪一列用于生成 HLL 列。它通常用来替代 Count Distinct，并且可以结合 rollup 快速计算业务中的 UV。

## 语法

```Haskell
HLL_UNION_AGG(hll)
```

## 示例

```plain
MySQL > select HLL_UNION_AGG(uv_set) from test_uv;
+-------------------------+
| HLL_UNION_AGG(`uv_set`) |
+-------------------------+
| 17721                   |
+-------------------------+
```

## 关键字

HLL_UNION_AGG, HLL, UNION, AGG