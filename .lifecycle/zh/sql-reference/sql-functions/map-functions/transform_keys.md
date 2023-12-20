---
displayed_sidebar: English
---

# 键值转换

## 描述

通过使用[Lambda expression](../Lambda_expression.md)转换映射中的键，并为映射中的每个条目产生一个新键。

该功能从 v3.1 版本开始提供支持。

## 语法

```Haskell
MAP transform_keys(lambda_func, any_map)
```

lambda_func 也可以放置在 any_map 之后：

```Haskell
MAP transform_keys(any_map, lambda_func)
```

## 参数

- any_map：映射对象。

- lambda_func：你想要应用到 any_map 上的 Lambda 表达式。

## 返回值

返回一个映射值，其键的数据类型由 Lambda 表达式的结果决定，而值的数据类型与 any_map 中的值保持一致。

如果任何输入参数为 NULL，就返回 NULL。

如果原始映射中的键或值为 NULL，那么 NULL 就作为一个正常的值来处理。

Lambda 表达式必须有两个参数。第一个参数代表键。第二个参数代表值。

## 示例

下面的示例使用 [map_from_arrays](map_from_arrays.md) 函数生成一个映射值 `{1:"ab", 3:"cdd", 2:null, null:"abc"}`。接着对每个键应用 Lambda 表达式，将每个键的值增加 1。

```SQL
mysql> select transform_keys((k,v)->(k+1), col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------+
| transform_keys((k, v) -> k + 1, col_map) |
+------------------------------------------+
| {2:"ab",4:"cdd",3:null,null:"abc"}       |
+------------------------------------------+
```
