---
displayed_sidebar: English
---

# distinct_map_keys

## 描述

从映射中删除重复的键，因为在语义上，映射中的键必须是唯一的。此函数仅保留相同键的最后一个值，称为 LAST WIN。在从外部表查询 MAP 数据时，如果映射中存在重复键，则会使用此函数。StarRocks 内部表会在地图中原生移除重复键。

此功能从 v3.1 开始支持。

## 语法

```Haskell
distinct_map_keys(any_map)
```

## 参数

`any_map`：要从中删除重复键的 MAP 值。

## 返回值

返回一个新的映射，其中每个映射中都没有重复的键。

如果输入为 NULL，则返回 NULL。

## 例子

示例 1：简单用法。

```plain
select distinct_map_keys(map{"a":1,"a":2});
+-------------------------------------+
| distinct_map_keys(map{'a':1,'a':2}) |
+-------------------------------------+
| {"a":2}                             |
+-------------------------------------+
```

示例 2：从外部表查询 MAP 数据，并从 `col_map` 列中删除重复的键。

```plain
select distinct_map_keys(col_map) as unique, col_map from external_table;
+---------------+---------------+
|      unique   | col_map       |
+---------------+---------------+
|       {"c":2} | {"c":1,"c":2} |
|           NULL|          NULL |
| {"e":4,"d":5} | {"e":4,"d":5} |
+---------------+---------------+
3 rows in set (0.05 sec)
```