---
displayed_sidebar: Chinese
---

# bitnot

## 機能

引数 `x` のビット反転演算の結果を返します。

## 文法

```Haskell
BITNOT(x);
```

## パラメータ説明

`x`: 対応するデータ型は TINYINT、SMALLINT、INT、BIGINT、LARGEINT です。

## 戻り値の説明

戻り値のデータ型は `x` の型と同じです。

## 例

```Plain Text
mysql> select bitnot(3);
+-----------+
| bitnot(3) |
+-----------+
|        -4 |
+-----------+
1行がセットされました (0.00 秒)
```
