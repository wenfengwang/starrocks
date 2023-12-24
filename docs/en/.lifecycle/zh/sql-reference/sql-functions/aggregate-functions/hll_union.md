---
displayed_sidebar: English
---


# hll_union

## 描述

返回一组 HLL 值的连接。

## 语法

```Haskell
hll_union(hll)
```

## 例子

```Plain
mysql> select k1, hll_cardinality(hll_union(v1)) from tbl group by k1;
+------+----------------------------------+
| k1   | hll_cardinality(hll_union(`v1`)) |
+------+----------------------------------+
|    2 |                                4 |
|    1 |                                3 |
+------+----------------------------------+