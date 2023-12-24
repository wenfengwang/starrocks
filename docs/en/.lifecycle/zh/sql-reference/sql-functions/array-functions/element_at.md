---
displayed_sidebar: English
---

# element_at

## 描述

从给定数组中返回指定位置（索引）处的元素。如果任何参数为 NULL 或位置不存在，则结果为 NULL。

此函数是下标运算符 `[]` 的别名。从 v3.0 版本开始支持该函数。

如果要从映射中的键值对中检索值，请参阅 [element_at](../map-functions/element_at.md)。

## 语法

```Haskell
element_at(any_array, position)
```

## 参数

- `any_array`：要检索元素的数组表达式。
- `position`：数组中的元素位置。必须为正整数。取值范围：[1, 数组长度]。如果 `position` 不存在，则返回 NULL。

## 例子

```plain text
mysql> select element_at([2,3,11],3);
+-----------------------+
|  element_at([11,2,3]) |
+-----------------------+
|                     11 |
+-----------------------+
1 行记录 (0.00 秒)
```

## 关键词

ELEMENT_AT、ARRAY