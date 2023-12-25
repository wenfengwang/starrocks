---
displayed_sidebar: Chinese
---

# negative

## 機能

引数 `x` の負数を結果として出力します。

## 文法

```Haskell
NEGATIVE(x);
```

## 引数説明

`x`: サポートされているデータ型は BIGINT、DOUBLE、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128 です。

## 戻り値の説明

戻り値のデータ型は引数 `x` の型と同じです。

## 例

```Plain Text
mysql>  select negative(3);
+-------------+
| negative(3) |
+-------------+
|          -3 |
+-------------+
1 row in set (0.00 sec)

mysql> select negative(cast(3.14 as decimalv2));
+--------------------------------------+
| negative(CAST(3.14 AS DECIMAL(9,0))) |
+--------------------------------------+
|                                -3.14 |
+--------------------------------------+
1 row in set (0.01 sec)
```
