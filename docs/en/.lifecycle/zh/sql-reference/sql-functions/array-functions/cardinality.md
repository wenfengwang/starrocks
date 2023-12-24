---
displayed_sidebar: English
---

# 基数

## 描述

返回数组中的元素数量。结果类型为 INT。如果输入参数为 NULL，则结果也为 NULL。Null 元素计入长度。

它是 [array_length() 的别名](array_length.md)。

此函数从 v3.0 版本开始支持。

## 语法

```Haskell
INT cardinality(any_array)
```

## 参数

`any_array`：要检索元素数量的 ARRAY 值。

## 返回值

返回一个 INT 值。

## 例子

```plain text
mysql> select cardinality([1,2,3]);
+-----------------------+
|  cardinality([1,2,3]) |
+-----------------------+
|                     3 |
+-----------------------+
1 行记录 (0.00 秒)

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
1 行记录 (0.01 秒)
```

## 关键词

基数、ARRAY_LENGTH、数组