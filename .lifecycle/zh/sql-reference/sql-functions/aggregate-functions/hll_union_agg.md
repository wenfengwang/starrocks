---
displayed_sidebar: English
---

# hll_union_agg

## 描述

HLL 是基于 HyperLogLog 算法的工程实现，用于储存 HyperLogLog 计算过程中的中间结果。

它只能作为表的值列使用，并通过聚合操作减少数据量，以此来加快查询速度。

基于 HLL 的估算结果误差大约为 1%。HLL 列可以由表中的其他列生成，或者基于载入到表中的数据生成。

在数据加载过程中，使用[hll_hash](../aggregate-functions/hll_hash.md)函数来指定哪一列用于生成HLL列。它常用来替代Count Distinct，并通过结合rollup，在商业分析中快速计算UVs。

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
