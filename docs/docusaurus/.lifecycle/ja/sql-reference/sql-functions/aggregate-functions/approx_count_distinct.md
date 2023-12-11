---
displayed_sidebar: "Japanese"
---

# approx_count_distinct

## 説明

COUNT(DISTINCT col)の結果に類似した集計関数の近似値を返します。

COUNTとDISTINCTの組み合わせよりも高速で、固定サイズのメモリを使用するため、高基数の列に対して少ないメモリが使用されます。

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