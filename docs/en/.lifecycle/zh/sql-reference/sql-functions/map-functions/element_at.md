---
displayed_sidebar: English
---

# element_at

## 描述

从地图的键值对中返回指定键的值。如果任何输入参数为 NULL，或者地图中不存在该键，则结果为 NULL。

如果要从数组中检索元素，请参见[element_at](../array-functions/element_at.md)。

此函数从 v3.0 版本开始支持。

## 语法

```Haskell
element_at(any_map, any_key)
```

## 参数

- `any_map`：要检索值的地图表达式。
- `any_key`：地图中的键。

## 返回值

如果 `any_key` 存在于 `any_map` 中，则返回与该键对应的值。否则，返回 NULL。

## 例子

```plain text
mysql> select element_at(map{1:3,2:4},1);
+-------------------------+
| element_at({1:3,2:4},1) |
+-------------------------+
|                     3   |
+-------------------------+

mysql> select element_at(map{1:3,2:4},3);
+-------------------------+
| element_at({1:3,2:4},3) |
+-------------------------+
|                    NULL |
+-------------------------+

mysql> select element_at(map{'a':1,'b':2},'a');
+-----------------------+
| map{'a':1,'b':2}['a'] |
+-----------------------+
|                     1 |
+-----------------------+
```

## 关键字

ELEMENT_AT、地图
