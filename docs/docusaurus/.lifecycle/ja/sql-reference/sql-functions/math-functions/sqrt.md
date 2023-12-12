---
displayed_sidebar: "Japanese"
---

# sqrt, dsqrt

## 説明

値の平方根を計算します。 dsqrtはsqrtと同じです。

## 構文

```Haskell
DOUBLE SQRT(DOUBLE x);
DOUBLE DSQRT(DOUBLE x);
```

## パラメータ

`x`: 数値のみを指定できます。この関数は計算前に数値をDOUBLE値に変換します。

## 戻り値

DOUBLEデータ型の値を返します。

## 使用上の注意

非数値を指定した場合、この関数は`NULL`を返します。

## 例

```Plain
mysql> select sqrt(3.14);
+-------------------+
| sqrt(3.14)        |
+-------------------+
| 1.772004514666935 |
+-------------------+
1 row in set (0.01 sec)


mysql> select dsqrt(3.14);
+-------------------+
| dsqrt(3.14)       |
+-------------------+
| 1.772004514666935 |
+-------------------+
```