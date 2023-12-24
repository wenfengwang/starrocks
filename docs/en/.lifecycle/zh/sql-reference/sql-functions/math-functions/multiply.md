---
displayed_sidebar: English
---

# 乘法

## 描述

计算参数的乘积。

## 语法

```Haskell
multiply(arg1, arg2)
```

### 参数

`arg1`：数字源列或文字值。
`arg2`：数字源列或文字值。

## 返回值

返回两个参数的乘积。返回类型取决于参数的类型。

## 使用说明

如果指定了非数值的值，此函数将返回错误。

## 例子

```Plain
MySQL [test]> select multiply(10,2);
+-----------------+
| multiply(10, 2) |
+-----------------+
|              20 |
+-----------------+
1 row in set (0.01 sec)

MySQL [test]> select multiply(1,2.1);
+------------------+
| multiply(1, 2.1) |
+------------------+
|              2.1 |
+------------------+
1 row in set (0.01 sec)

MySQL [test]> select * from t;
+------+------+------+------+
| id   | name | job1 | job2 |
+------+------+------+------+
|    2 |    2 |    2 |    2 |
+------+------+------+------+
1 row in set (0.08 sec)

MySQL [test]> select multiply(1.0,id) from t;
+-------------------+
| multiply(1.0, id) |
+-------------------+
|                 2 |
+-------------------+
1 row in set (0.01 sec)
```

## 关键词

乘法