---
displayed_sidebar: English
---

# 数组基数

## 描述

返回数组中元素的个数。返回结果的类型为 INT。如果输入参数为 NULL，那么返回结果也是 NULL。数组中的 NULL 元素也会被计入总长度。

它是[array_length()](array_length.md)函数的同义词。

该函数从 v3.0 版本开始提供支持。

## 语法

```Haskell
INT cardinality(any_array)
```

## 参数

any_array：你想要获取元素个数的 ARRAY 类型的值。

## 返回值

返回一个 INT 类型的值。

## 示例

```plain
mysql> select cardinality([1,2,3]);
+-----------------------+
|  cardinality([1,2,3]) |
+-----------------------+
|                     3 |
+-----------------------+
1 row in set (0.00 sec)

mysql> select cardinality([1,2,3,null]);
+------------------------------+
| cardinality([1, 2, 3, NULL]) |
+------------------------------+
|                            4 |
+------------------------------+

mysql> select cardinality([[1,2], [3,4]]);
+-----------------------------+
|  cardinality([[1,2],[3,4]]) |
+-----------------------------+
|                           2 |
+-----------------------------+
1 row in set (0.01 sec)
```

## 关键词

CARDINALITY, ARRAY_LENGTH, ARRAY
