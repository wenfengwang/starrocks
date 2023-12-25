---
displayed_sidebar: Chinese
---

# truncate

## 機能

数値 `x` を小数点以下 `y` 桁で切り捨てた値を返します。

## 文法

```Haskell
TRUNCATE(x,y);
```

## 引数説明

`x`: 対応するデータ型は DOUBLE または DECIMAL128 です。

`y`: 対応するデータ型は INT です。

## 戻り値の説明

戻り値のデータ型は `x` と同じです。

## 例

```Plain Text
mysql> select truncate(3.14,1);
+-------------------+
| truncate(3.14, 1) |
+-------------------+
|               3.1 |
+-------------------+
1 row in set (0.00 sec)
```
