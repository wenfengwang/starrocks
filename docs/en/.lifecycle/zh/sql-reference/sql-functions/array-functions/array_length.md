---
displayed_sidebar: English
---

# array_length

## 描述

返回数组中的元素数。结果类型为 INT。如果输入参数为 NULL，则结果也为 NULL。空元素也计入长度。

它有一个别名 [cardinality()](cardinality.md)。

## 语法

```Haskell
INT array_length(any_array)
```

## 参数

`any_array`：要检索元素数的 ARRAY 值。

## 返回值

返回一个 INT 值。

## 例子

```plain text
mysql> select array_length([1,2,3]);
+-----------------------+
| array_length([1,2,3]) |
+-----------------------+
|                     3 |
+-----------------------+
1 行记录 (0.00 秒)

mysql> select array_length([1,2,3,null]);
+-------------------------------+
| array_length([1, 2, 3, NULL]) |
+-------------------------------+
|                             4 |
+-------------------------------+

mysql> select array_length([[1,2], [3,4]]);
+-----------------------------+
| array_length([[1,2],[3,4]]) |
+-----------------------------+
|                           2 |
+-----------------------------+
1 行记录 (0.01 秒)
```

## 关键字

ARRAY_LENGTH、数组、基数