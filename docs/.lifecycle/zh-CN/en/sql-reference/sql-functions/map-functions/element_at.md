---
displayed_sidebar: "Chinese"
---

# element_at

## 描述

从键值对的地图中返回指定键的值。如果任何输入参数为NULL或者地图中不存在该键，则结果为NULL。

如果您想要从数组中检索元素，请参见[element_at](../array-functions/element_at.md)。

此功能从v3.0版本开始受支持。

## 语法

```Haskell
element_at(any_map, any_key)
```

## 参数

- `any_map`: 用于检索值的地图表达式。
- `any_key`: 地图中的键。

## 返回值

如果`any_key`存在于`any_map`中，则返回与该键对应的值。否则，返回NULL。

## 示例

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

## 关键词

ELEMENT_AT, MAP