---
displayed_sidebar: English
---

# ELEMENT_AT 函数

## 描述

返回指定数组中特定位置（索引）上的元素。如果任何参数为 NULL 或者指定位置不存在，结果将返回 NULL。

此函数是下标运算符[]的别名。从 v3.0 版本开始提供支持。

若需从映射（map）中的键值对提取值，请参考[element_at](../map-functions/element_at.md)函数。

## 语法

```Haskell
element_at(any_array, position)
```

## 参数

- any_array：用于检索元素的数组表达式。
- position：数组中元素的位置。必须是正整数。取值范围：[1，数组长度]。如果指定位置不存在，将返回 NULL。

## 示例

```plain
mysql> select element_at([2,3,11],3);
+-----------------------+
|  element_at([11,2,3]) |
+-----------------------+
|                     11 |
+-----------------------+
1 row in set (0.00 sec)
```

## 关键字

ELEMENT_AT, ARRAY
