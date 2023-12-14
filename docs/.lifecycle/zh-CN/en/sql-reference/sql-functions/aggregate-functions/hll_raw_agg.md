---
displayed_sidebar: "中文"
---

# hll_raw_agg

## 描述

此函数是一个聚合函数，用于聚合HLL字段。 它返回一个HLL值。

## 语法

```Haskell
hll_raw_agg(hll)
```

## 参数

`hll`: 由其他列生成或基于加载数据生成的HLL列。

## 返回值

返回一个HLL类型的值。

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