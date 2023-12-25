---
displayed_sidebar: Chinese
---

# BITOR

## 機能

二つの数値に対してビット単位の OR 演算を行った結果を返します。

## 文法

```Haskell
BITOR(x,y);
```

## 引数説明

`x`: 対応するデータ型は TINYINT、SMALLINT、INT、BIGINT、LARGEINT です。

`y`: 対応するデータ型は TINYINT、SMALLINT、INT、BIGINT、LARGEINT です。

> 注: 使用する際には `x` と `y` のデータ型が同じである必要があります。

## 戻り値の説明

戻り値のデータ型は `x` の型と一致します。

## 例

```Plain Text
mysql> select BITOR(3,0);
+-------------+
| BITOR(3, 0) |
+-------------+
|           3 |
+-------------+
1 row in set (0.00 sec)
```
