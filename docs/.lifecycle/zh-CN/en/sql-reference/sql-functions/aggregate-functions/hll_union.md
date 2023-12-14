---
displayed_sidebar: "Chinese"
---


# hll_union

## 描述

返回一组HLL值的串联。

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