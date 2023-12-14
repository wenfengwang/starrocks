---
displayed_sidebar: "中文"
---

# array_contains_seq

## 描述

检查array2的所有元素是否按相同的顺序出现在array1中。因此，只有当array1 = prefix + array2 + suffix时，函数才会返回1。

## 语法

~~~Haskell
BOOLEAN array_contains_all(arr1, arr2)
~~~

## 参数

`arr`：要比较的两个数组。此语法检查`arr2`是否是`arr1`的子集且顺序完全相同。

两个数组元素的数据类型必须相同。有关StarRocks支持的数组元素数据类型，请参见[ARRAY](../../../sql-reference/sql-statements/data-types/Array.md)。

## 返回值

返回BOOLEAN类型的值。

如果`arr2`是`arr1`的子集，则返回1。否则返回0。
将Null视为一个值。换句话说，array_contains_seq([1, 2, NULL, 3, 4], [2, 3])将返回0。然而，array_contains_seq([1, 2, NULL, 3, 4], [2, NULL, 3])将返回1。
两个数组中值的顺序是重要的。

## 示例

返回BOOLEAN类型的值。

```Plaintext
MySQL [(none)]> select array_contains_seq([1,2,3,4], [1,2,3]);
+---------------------------------------------+
| array_contains_seq([1, 2, 3, 4], [1, 2, 3]) |
+---------------------------------------------+
|                                           1 |
+---------------------------------------------+
```

```Plaintext
MySQL [(none)]> select array_contains_seq([1,2,3,4], [3,2]);
+------------------------------------------+
| array_contains_seq([1, 2, 3, 4], [3, 2]) |
+------------------------------------------+
|                                        0 |
+------------------------------------------+
1 row in set (0.18 sec)
```

```Plaintext
MySQL [(none)]> select array_contains_all([1, 2, NULL, 3, 4], ['a']);
+-----------------------------------------------+
| array_contains_all([1, 2, NULL, 3, 4], ['a']) |
+-----------------------------------------------+
|                                             0 |
+-----------------------------------------------+
1 row in set (0.18 sec)
```

```Plaintext
MySQL [(none)]> select array_contains([1, 2, NULL, 3, 4], 'a');
+-----------------------------------------+
| array_contains([1, 2, NULL, 3, 4], 'a') |
+-----------------------------------------+
|                                       0 |
+-----------------------------------------+
1 row in set (0.18 sec)
```
```Plaintext
MySQL [(none)]> SELECT array_contains([1, 2,3,4,null], null);
+------------------------------------------+
| array_contains([1, 2, 3, 4, NULL], NULL) |
+------------------------------------------+
|                                        1 |
+------------------------------------------+
1 row in set (0.18 sec)
```