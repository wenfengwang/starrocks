---
displayed_sidebar: English
---


# hll_union

## 描述

返回一组 HLL 值的合并结果。

## 语法

```Haskell
hll_union(hll)
```

## 示例

```Plain
mysql> select k1, hll_cardinality(hll_union(v1)) from tbl group by k1;
+------+----------------------------------+
| k1   | hll_cardinality(hll_union(`v1`)) |
+------+----------------------------------+
|    2 |                                4 |
|    1 |                                3 |
+------+----------------------------------+
```
