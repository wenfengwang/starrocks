---
displayed_sidebar: English
---

# array_contains_seq

## 描述

检查 array2 的所有元素是否以完全相同的顺序出现在 array1 中。因此，当且仅当 array1 = prefix + array2 + suffix 时，函数才会返回 1。

## 语法

```Haskell
BOOLEAN array_contains_seq(arr1, arr2)
```

## 参数

`arr`: 要比较的两个数组。此语法检查 `arr2` 是否是 `arr1` 的子集且顺序完全相同。

两个数组中元素的数据类型必须相同。有关 StarRocks 支持的数组元素数据类型，请参见 [ARRAY](../../../sql-reference/sql-statements/data-types/Array.md)。

## 返回值

返回 BOOLEAN 类型的值。

如果 `arr2` 是 `arr1` 的子集，则返回 1。否则，返回 0。Null 被当作一个值处理。换句话说，array_contains_seq([1, 2, NULL, 3, 4], [2,3]) 将返回 0。然而，array_contains_seq([1, 2, NULL, 3, 4], [2,NULL,3]) 将返回 1。两个数组中值的顺序是重要的。

## 示例

返回 BOOLEAN 类型的值。

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
MySQL [(none)]> select array_contains_seq([1, 2, NULL, 3, 4], ['a']);
+-----------------------------------------------+
| array_contains_seq([1, 2, NULL, 3, 4], ['a']) |
+-----------------------------------------------+
|                                             0 |
+-----------------------------------------------+
1 row in set (0.18 sec)
```

```Plaintext
MySQL [(none)]> select array_contains_seq([1, 2, NULL, 3, 4], 'a');
+-----------------------------------------+
| array_contains_seq([1, 2, NULL, 3, 4], 'a') |
+-----------------------------------------+
|                                       0 |
+-----------------------------------------+
1 row in set (0.18 sec)
```
```Plaintext
MySQL [(none)]> SELECT array_contains_seq([1, 2, 3, 4, NULL], NULL);
+------------------------------------------+
| array_contains_seq([1, 2, 3, 4, NULL], NULL) |
+------------------------------------------+
|                                        1 |
+------------------------------------------+
1 row in set (0.18 sec)
```