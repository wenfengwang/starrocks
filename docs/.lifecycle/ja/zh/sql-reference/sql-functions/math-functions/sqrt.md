---
displayed_sidebar: Chinese
---

# sqrt、dsqrt

## 機能

引数 `x` の平方根を返します。dsqrt() と sqrt() の機能は同じです。

## 文法

```Haskell
DOUBLE SQRT(DOUBLE x);
DOUBLE DSQRT(DOUBLE x);
```

## 引数説明

`x`: サポートされるデータ型は DOUBLE です。

## 戻り値の説明

戻り値のデータ型は DOUBLE です。

## 例

```Plain Text
mysql> select sqrt(3.14);
+-------------------+
| sqrt(3.14)        |
+-------------------+
| 1.772004514666935 |
+-------------------+
1行がセットされました (0.01 sec)


mysql> select dsqrt(3.14);
+-------------------+
| dsqrt(3.14)       |
+-------------------+
| 1.772004514666935 |
+-------------------+
```
