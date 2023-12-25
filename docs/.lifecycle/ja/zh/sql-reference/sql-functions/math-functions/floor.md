---
displayed_sidebar: Chinese
---

# floor, dfloor

## 機能

`x` 以下の最大整数値を返します。

## 文法

```Haskell
FLOOR(x);
```

## パラメータ説明

`x`: 対応するデータ型は DOUBLE です。

## 戻り値の説明

戻り値のデータ型は BIGINT です。

## 例

```Plain Text
mysql> select floor(3.14);
+-------------+
| floor(3.14) |
+-------------+
|           3 |
+-------------+
1行がセットされました (0.01 秒)
```
