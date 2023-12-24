---
displayed_sidebar: English
---

# approx_count_distinct

## 描述

返回聚合函数的近似值，类似于 COUNT(DISTINCT col) 的结果。

它比 COUNT 和 DISTINCT 的组合更快，并且使用固定大小的内存，因此在高基数列上使用的内存较少。

## 语法

```Haskell
APPROX_COUNT_DISTINCT(expr)
```

## 例子

```plain text
MySQL > select approx_count_distinct(query_id) from log_statis group by datetime;
+-----------------------------------+
| approx_count_distinct(`query_id`) |
+-----------------------------------+
| 17721                             |
+-----------------------------------+
```

## 关键词

APPROX_COUNT_DISTINCT
