---
displayed_sidebar: English
---

# multiply

## 描述

计算参数的乘积。

## 语法

```Haskell
multiply(arg1, arg2)
```

### 参数

`arg1`：数值型源列或字面量。
`arg2`：数值型源列或字面量。

## 返回值

返回两个参数的乘积。返回类型依据参数而定。

## 使用说明

如果指定了非数值型值，此函数将执行失败。

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

multiply