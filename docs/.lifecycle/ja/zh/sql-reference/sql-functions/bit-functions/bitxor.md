---
displayed_sidebar: Chinese
---

# bitxor

## 機能

二つの数値に対するビット単位のXOR演算の結果を返します。

## 文法

```Haskell
BITXOR(x,y);
```

## パラメータ説明

`x`: 対応するデータ型は TINYINT、SMALLINT、INT、BIGINT、LARGEINT です。

`y`: 対応するデータ型は TINYINT、SMALLINT、INT、BIGINT、LARGEINT です。

> 注：使用する際には `x` と `y` のデータ型が同じである必要があります。

## 戻り値の説明

戻り値のデータ型は `x` の型と同じです。

## 例

```Plain Text
mysql> select bitxor(3,0);
+--------------+
| bitxor(3, 0) |
+--------------+
|            3 |
+--------------+
1行がセットされました (0.00 sec)
```
