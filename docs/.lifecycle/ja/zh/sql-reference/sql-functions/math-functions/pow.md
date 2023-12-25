---
displayed_sidebar: Chinese
---

# pow、power、dpow、fpow

## 機能

引数 `x` の `y` 乗を返します。

## 文法

```Haskell
POW(x,y);
POWER(x,y);
```

## 引数説明

`x`: サポートされているデータ型は DOUBLE です。

`y`: サポートされているデータ型は DOUBLE です。

## 戻り値の説明

戻り値のデータ型は DOUBLE です。

## 例

```Plain Text
mysql> select pow(2,2);
+-----------+
| pow(2, 2) |
+-----------+
|         4 |
+-----------+
1 row in set (0.00 sec)

mysql> select power(4,3);
+-------------+
| power(4, 3) |
+-------------+
|          64 |
+-------------+
1 row in set (0.00 sec)

```
