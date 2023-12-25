---
displayed_sidebar: Chinese
---

# approx_count_distinct

## 機能

`COUNT(DISTINCT col)` の結果に似た近似値を返します。

> COUNTとDISTINCTの組み合わせよりも速く、固定サイズのメモリを使用するため、高カーディナリティの列に対してはより少ないメモリを使用できます。

## 文法

```Haskell
APPROX_COUNT_DISTINCT(expr)
```

## パラメータ説明

`expr`: 選択された表現。

## 戻り値の説明

数値型の戻り値です。

## 例

```plain text
MySQL > select approx_count_distinct(query_id) from log_statis group by datetime;
+-----------------------------------+
| approx_count_distinct(`query_id`) |
+-----------------------------------+
| 17721                             |
+-----------------------------------+
```
