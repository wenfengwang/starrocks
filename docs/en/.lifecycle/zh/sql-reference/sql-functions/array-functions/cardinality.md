---
displayed_sidebar: English
---

# cardinality

## 描述

返回数组中的元素数量。结果类型为 INT。如果输入参数为 NULL，则结果也为 NULL。NULL 元素也会被计入长度。

它是 [array_length()](array_length.md) 的别名。

该函数从 v3.0 版本开始支持。

## 语法

```Haskell
INT cardinality(any_array)
```

## 参数

`any_array`：你想要获取元素数量的 ARRAY 值。

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

mysql> select cardinality([1,2,3,NULL]);
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

## 关键字

CARDINALITY, ARRAY_LENGTH, ARRAY