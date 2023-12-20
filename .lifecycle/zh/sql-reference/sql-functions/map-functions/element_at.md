---
displayed_sidebar: English
---

# 元素_at

## 描述

从映射的键值对中返回指定键对应的值。如果任何输入参数为 NULL 或者映射中不存在该键，结果将返回 NULL。

如果您希望从数组中提取元素，请参考[element_at](../array-functions/element_at.md)函数。

该功能从 v3.0 版本开始提供支持。

## 语法

```Haskell
element_at(any_map, any_key)
```

## 参数

- any_map：用于从中检索值的 MAP 表达式。
- any_key：映射中的一个键。

## 返回值

如果 any_map 中存在 any_key，将返回与该键对应的值。否则，返回 NULL。

## 示例

```plain
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
