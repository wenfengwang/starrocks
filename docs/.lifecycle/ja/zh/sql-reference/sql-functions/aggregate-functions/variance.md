---
displayed_sidebar: Chinese
---

# 分散、var_pop、variance_pop

## 機能

式の母集団分散を返します。バージョン2.5.10から、この関数はウィンドウ関数としても使用できます。

## 文法

```Haskell
VARIANCE(expr)
```

## パラメータ説明

`expr`: 選択された式。式がテーブルの列の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL。

## 戻り値の説明

DOUBLE型の値を返します。

## 例

```plaintext
select var_pop(i_current_price), i_rec_start_date from item group by i_rec_start_date;
+--------------------------+------------------+
| var_pop(i_current_price) | i_rec_start_date |
+--------------------------+------------------+
|       314.96177792808226 | 1997-10-27       |
|       463.73633459357285 | NULL             |
|       302.02102643609123 | 1999-10-28       |
|        337.9318386924913 | 2000-10-27       |
|       333.80931439318346 | 2001-10-27       |
+--------------------------+------------------+

select variance(i_current_price), i_rec_start_date from item group by i_rec_start_date;
+---------------------------+------------------+
| variance(i_current_price) | i_rec_start_date |
+---------------------------+------------------+
|        314.96177792808226 | 1997-10-27       |
|         463.7363345935729 | NULL             |
|        302.02102643609123 | 1999-10-28       |
|         337.9318386924912 | 2000-10-27       |
|        333.80931439318346 | 2001-10-27       |
+---------------------------+------------------+
```
