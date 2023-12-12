---
displayed_sidebar: "Japanese"
---

# approx_count_distinct

## Description

COUNT(DISTINCT col)の結果に類似した集計関数の近似値を返します。

COUNTとDISTINCTの組み合わせよりも速く、固定サイズのメモリを使用するため、高い基数の列のためにより少ないメモリを使用します。

## Syntax

```Haskell
APPROX_COUNT_DISTINCT(expr)
```

## Examples

```plain text
MySQL > select approx_count_distinct(query_id) from log_statis group by datetime;
+-----------------------------------+
| approx_count_distinct(`query_id`) |
+-----------------------------------+
| 17721                             |
+-----------------------------------+
```

## keyword

APPROX_COUNT_DISTINCT