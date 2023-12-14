---
displayed_sidebar: "Chinese"
---

# element_at

## 描述 

返回给定数组中指定位置（索引）的元素。如果任何参数为NULL或位置不存在，则结果为NULL。

该函数是下标运算符`[]`的别名。从v3.0版本开始支持。

如果你想从地图中的键值对中检索值, 请参阅[element_at](../map-functions/element_at.md)。

## 语法

```Haskell
element_at(any_array, position)
```

## 参数

- `any_array`：要检索元素的数组表达式。
- `position`：数组中元素的位置。必须是正整数。值范围：[1，数组长度]。如果`position`不存在，则返回NULL。

## 示例

```plain text
mysql> select element_at([2,3,11],3);
+-----------------------+
|  element_at([11,2,3]) |
+-----------------------+
|                     11 |
+-----------------------+
1 row in set (0.00 sec)
```

## 关键词

ELEMENT_AT, ARRAY