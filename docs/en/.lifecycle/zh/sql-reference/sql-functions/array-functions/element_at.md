---
displayed_sidebar: English
---

# element_at

## 描述

返回给定数组中指定位置（索引）的元素。如果任何参数为 NULL 或位置不存在，则结果为 NULL。

此函数是下标运算符 `[]` 的别名。从 v3.0 版本开始支持。

如果您想从映射中的键值对检索值，请参阅 [element_at](../map-functions/element_at.md)。

## 语法

```Haskell
element_at(any_array, position)
```

## 参数

- `any_array`：用于检索元素的数组表达式。
- `position`：元素在数组中的位置。它必须是一个正整数。值范围：[1, 数组长度]。如果 `position` 不存在，则返回 NULL。

## 示例

```plain
mysql> select element_at([2,3,11],3);
+-----------------------+
|  element_at([2,3,11]) |
+-----------------------+
|                     11 |
+-----------------------+
1 row in set (0.00 sec)
```

## 关键字

ELEMENT_AT, ARRAY