---
displayed_sidebar: "中文"
---

# map_apply

## 描述

将Lambda表达式应用于原始映射的键和值，并生成新的映射。此功能支持v3.0及以上。

## 语法

```Haskell
MAP map_apply(lambda_func, any_map)
```

## 参数

- `lambda_func`：Lambda表达式。

- `any_map`：应用Lambda表达式的映射。

## 返回值

返回一个映射值。结果映射中键和值的数据类型由Lambda表达式的结果确定。

如果任何输入参数为NULL，则返回NULL。

如果原始映射中的键或值为NULL，则将NULL作为正常值处理。

Lambda表达式必须有两个参数。第一个参数表示键，第二个参数表示值。

## 示例

以下示例使用[map_from_arrays()](map_from_arrays.md)生成一个映射值 `{1:"ab",3:"cd"}`。然后Lambda表达式将每个键增加1，并计算每个值的长度。例如，"ab"的长度为2。

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
```