---
displayed_sidebar: English
---

# approx_count_distinct

## 描述

返回一个聚合函数的近似值，该函数与 COUNT(DISTINCT col) 的结果类似。

它比 COUNT 和 DISTINCT 的组合运算更快，并且使用固定大小的内存，所以对于高基数的列，其使用的内存更少。

## 语法

```Haskell
APPROX_COUNT_DISTINCT(expr)
```

## 示例

```plain
MySQL > select approx_count_distinct(query_id) from log_statis group by datetime;
+-----------------------------------+
| approx_count_distinct(`query_id`) |
+-----------------------------------+
| 17721                             |
+-----------------------------------+
```

## 关键字

APPROX_COUNT_DISTINCT