---
displayed_sidebar: "Japanese"
---

# sqrt, dsqrt（ルート算出）

## Description（概要）

値の平方根を計算します。dsqrtはsqrtと同じです。

## Syntax（構文）

```Haskell
DOUBLE SQRT(DOUBLE x);
DOUBLE DSQRT(DOUBLE x);
```

## Parameters（パラメータ）

`x`: 数値のみを指定できます。この関数は計算前に数値をDOUBLE値に変換します。

## Return value（戻り値）

DOUBLEデータ型の値を返します。

## Usage notes（使用上の注意）

数値以外を指定すると、この関数は `NULL` を返します。

## Examples（例）

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