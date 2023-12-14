---
displayed_sidebar: "Chinese"
---

# approx_count_distinct

## 描述

返回类似于COUNT(DISTINCT col)结果的聚合函数的近似值。

它比COUNT和DISTINCT组合更快，使用固定大小的内存，因此对于高基数的列使用的内存较少。

## 语法

```Haskell
APPROX_COUNT_DISTINCT(expr)
```

## 示例

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