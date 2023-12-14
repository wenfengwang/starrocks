---
displayed_sidebar: "Chinese"
---

# pow, power, dpow, fpow

## 描述

返回 `x` 的 `y` 次方的结果。

## 语法

```Haskell
POW(x,y);POWER(x,y);
```

## 参数

`x`: 支持 DOUBLE 数据类型。

`y`: 支持 DOUBLE 数据类型。

## 返回值

返回一个 DOUBLE 数据类型的值。

## 示例

```Plain
mysql> select pow(2,2);
+-----------+
| pow(2, 2) |
+-----------+
|         4 |
+-----------+
1 行受到影响 (0.00 sec)

mysql> select power(4,3);
+-------------+
| power(4, 3) |
+-------------+
|          64 |
+-------------+
1 行受到影响 (0.00 sec)
```