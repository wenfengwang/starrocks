---
displayed_sidebar: English
---

# hll_raw_agg

## 描述

此函数是一个聚合函数，用于聚合 HLL 字段。它返回一个 HLL 值。

## 语法

```Haskell
hll_raw_agg(hll)
```

## 参数

`hll`：由其他列生成或基于加载数据生成的 HLL 列。

## 返回值

返回一个 HLL 类型的值。

## 例子

```Plain
mysql> select k1, hll_cardinality(hll_raw_agg(v1)) from tbl group by k1;
+------+----------------------------------+
| k1   | hll_cardinality(hll_raw_agg(`v1`)) |
+------+----------------------------------+
|    2 |                                4 |
|    1 |                                3 |
+------+----------------------------------+