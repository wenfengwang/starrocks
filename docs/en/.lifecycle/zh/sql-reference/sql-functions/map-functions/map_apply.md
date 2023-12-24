---
displayed_sidebar: English
---

# map_apply

## 描述

将 [Lambda 表达式](../Lambda_expression.md) 应用于原始 Map 的键和值，并生成新的 Map。从 v3.0 开始支持此功能。

## 语法

```Haskell
MAP map_apply(lambda_func, any_map)
```

## 参数

- `lambda_func`：Lambda 表达式。

- `any_map`：要应用 Lambda 表达式的映射。

## 返回值

返回一个映射值。结果映射中的键和值的数据类型由 Lambda 表达式的结果确定。

如果任何输入参数为 NULL，则返回 NULL。

如果原始映射中的键或值为 NULL，则将 NULL 作为正常值进行处理。

Lambda 表达式必须具有两个参数。第一个参数表示键，第二个参数表示值。

## 例子

以下示例使用 [map_from_arrays()](map_from_arrays.md) 生成映射值 `{1:"ab",3:"cd"}`。然后，Lambda 表达式递增每个键 1 并计算每个值的长度。例如，“ab”的长度为 2。

```SQL
mysql> select map_apply((k,v)->(k+1,length(v)), col_map)
from (select map_from_arrays([1,3],["ab","cd"]) as col_map)A;
+--------------------------------------------------+
| map_apply((k, v) -> (k + 1, length(v)), col_map) |
+--------------------------------------------------+
| {2:2,4:2}                                        |
+--------------------------------------------------+
1 row in set (0.01 sec)

mysql> select map_apply((k,v)->(k+1,length(v)), col_map)
from (select map_from_arrays(null,null) as col_map)A;
+--------------------------------------------------+
| map_apply((k, v) -> (k + 1, length(v)), col_map) |
+--------------------------------------------------+
| NULL                                             |
+--------------------------------------------------+
1 row in set (0.02 sec)

mysql> select map_apply((k,v)->(k+1,length(v)), col_map)
from (select map_from_arrays([1,null],["ab","cd"]) as col_map)A;
+--------------------------------------------------+
| map_apply((k, v) -> (k + 1, length(v)), col_map) |
+--------------------------------------------------+
| NULL                                             |
+--------------------------------------------------+