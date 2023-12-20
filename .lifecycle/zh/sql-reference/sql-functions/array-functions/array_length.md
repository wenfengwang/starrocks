---
displayed_sidebar: English
---

# 数组长度

## 描述

返回数组中元素的个数。返回结果的类型是 INT。如果输入参数是 NULL，那么返回结果也是 NULL。数组中的空元素也会被计入总长度。

它有一个别名[cardinality()](cardinality.md)。

## 语法

```Haskell
INT array_length(any_array)
```

## 参数

any_array：你想要获取元素数量的 ARRAY 值。

## 返回值

返回一个 INT 类型的值。

## 示例

```plain
mysql> select array_length([1,2,3]);
+-----------------------+
| array_length([1,2,3]) |
+-----------------------+
|                     3 |
+-----------------------+
1 row in set (0.00 sec)

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
1 row in set (0.01 sec)
```

## 关键词

ARRAY_LENGTH, ARRAY, CARDINALITY
