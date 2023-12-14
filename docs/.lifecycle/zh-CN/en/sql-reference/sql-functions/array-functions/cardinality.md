---
displayed_sidebar: "Chinese"
---

# cardinality

## 描述

返回数组中的元素数。 结果类型为INT。 如果输入参数为NULL，则结果也为NULL。 NULL元素在长度中被计算。

它是 [array_length()](array_length.md) 的别名。

此功能从v3.0开始受支持。

## 语法

```Haskell
INT cardinality(any_array)
```

## 参数

`any_array`：要检索元素数量的ARRAY值。

## 返回值

返回一个INT值。

## 示例

```plain text
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