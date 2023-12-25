---
displayed_sidebar: English
---

# approx_count_distinct

## 説明

COUNT(DISTINCT col) の結果に近い集約関数の近似値を返します。

COUNT と DISTINCT を組み合わせるよりも速く、固定サイズのメモリを使用するため、高カーディナリティの列に対して使用されるメモリが少なくて済みます。

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
