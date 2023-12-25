---
displayed_sidebar: Chinese
---

# bitand

## 機能

二つの数値をビット単位でAND演算した結果を返します。

## 文法

```Haskell
BITAND(x,y);
```

## 引数説明

`x`: 対応するデータ型は TINYINT、SMALLINT、INT、BIGINT、LARGEINT です。

`y`: 対応するデータ型は TINYINT、SMALLINT、INT、BIGINT、LARGEINT です。

> 注:使用する際には `x` と `y` のデータ型が一致している必要があります。

## 戻り値の説明

戻り値のデータ型は `x` の型と一致します。

## 例

```Plain Text
mysql> select bitand(3,0);
+--------------+
| bitand(3, 0) |
+--------------+
|            0 |
+--------------+
1行がセットされました (0.01 sec)
```
