---
displayed_sidebar: "Chinese"
---

# array_length

## 描述

返回数组中元素的数量。结果类型为INT。如果输入参数为NULL，则结果也为NULL。长度中计算了NULL元素。

它还有一个别名[cardinality()](cardinality.md)。

## 语法

```Haskell
INT array_length(any_array)
```

## 参数

`any_array`: 要检索元素数量的ARRAY值。

## 返回值

返回一个INT值。

## 例子

```plain text
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