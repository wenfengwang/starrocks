---
displayed_sidebar: English
---

# hll_raw_agg

## 描述

此函数为聚合函数，用于对 HLL 字段进行聚合操作。其返回值为 HLL 类型。

## 语法

```Haskell
hll_raw_agg(hll)
```

## 参数

hll：由其他列生成或基于已加载数据生成的 HLL 列。

## 返回值

返回一个 HLL 类型的值。

## 示例

```Plain
mysql> select k1, hll_cardinality(hll_raw_agg(v1)) from tbl group by k1;
+------+----------------------------------+
| k1   | hll_cardinality(hll_raw_agg(`v1`)) |
+------+----------------------------------+
|    2 |                                4 |
|    1 |                                3 |
+------+----------------------------------+
```
