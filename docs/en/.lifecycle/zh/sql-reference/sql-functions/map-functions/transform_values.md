---
displayed_sidebar: English
---

# transform_values

## 描述

使用 [Lambda expression](../Lambda_expression.md) 转换映射中的值，并为映射中的每个条目生成一个新值。

此函数从 v3.1 版本开始支持。

## 语法

```Haskell
MAP transform_values(lambda_func, any_map)
```

`lambda_func` 也可以放置在 `any_map` 之后：

```Haskell
MAP transform_values(any_map, lambda_func)
```

## 参数

- `any_map`：映射。

- `lambda_func`：你想应用到 `any_map` 的 Lambda 表达式。

## 返回值

返回一个映射值，其中值的数据类型由 Lambda 表达式的结果决定，键的数据类型与 `any_map` 中的键相同。

如果任何输入参数为 NULL，则返回 NULL。

如果原始映射中的键或值为 NULL，则将 NULL 作为正常值处理。

Lambda 表达式必须有两个参数。第一个参数代表键。第二个参数代表值。

## 示例

以下示例使用 [map_from_arrays](map_from_arrays.md) 生成映射值 `{1:"ab",3:"cdd",2:null,null:"abc"}`。然后将 Lambda 表达式应用于映射的每个值。第一个示例将每个键值对的值更改为 1。第二个示例将每个键值对的值更改为 null。

```SQL
mysql> select transform_values((k,v)->1, col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+----------------------------------------+
| transform_values((k, v) -> 1, col_map) |
+----------------------------------------+
| {1:1,3:1,2:1,null:1}                   |
+----------------------------------------+
1 row in set (0.02 sec)

mysql> select transform_values((k,v)->null, col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+--------------------------------------------+
| transform_values((k, v) -> NULL, col_map)  |
+--------------------------------------------+
| {1:null,3:null,2:null,null:null}           |
+--------------------------------------------+
1 row in set (0.01 sec)
```