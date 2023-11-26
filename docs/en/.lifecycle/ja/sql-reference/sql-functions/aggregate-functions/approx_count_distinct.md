---
displayed_sidebar: "Japanese"
---

# APPROX_COUNT_DISTINCT

## 説明

COUNT(DISTINCT col)の結果に類似した集計関数の近似値を返します。

COUNTとDISTINCTの組み合わせよりも高速であり、固定サイズのメモリを使用するため、高基数の列に対してはより少ないメモリを使用します。

## 構文

```Haskell
APPROX_COUNT_DISTINCT(expr)
```

## 例

```plain text
MySQL > select approx_count_distinct(query_id) from log_statis group by datetime;
+-----------------------------------+
| approx_count_distinct(`query_id`) |
+-----------------------------------+
| 17721                             |
+-----------------------------------+
```

## キーワード

APPROX_COUNT_DISTINCT
