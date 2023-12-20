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

arg1：数值型源列或字面量。arg2：数值型源列或字面量。

## 返回值

返回两个参数的乘积。返回的数据类型取决于参数。

## 使用须知

如果输入的是非数值类型的数据，该函数将无法执行。

## 示例

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

## 关键字

乘法
