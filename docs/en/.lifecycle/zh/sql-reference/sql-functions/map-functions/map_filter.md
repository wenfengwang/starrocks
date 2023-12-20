---
displayed_sidebar: English
---

# map_filter

## 描述

通过对映射中的每个键值对应用布尔数组或[Lambda 表达式](../Lambda_expression.md)来过滤键值对。返回那些计算结果为 `true` 的键值对。

该函数从 v3.1 版本开始支持。

## 语法

```Haskell
MAP map_filter(any_map, array<boolean>)
MAP map_filter(lambda_func, any_map)
```

- `map_filter(any_map, array<boolean>)`

  对 `any_map` 中的键值对逐一进行 `array<boolean>` 的评估，并返回那些评估结果为 `true` 的键值对。

- `map_filter(lambda_func, any_map)`

  对 `any_map` 中的键值对逐一应用 `lambda_func`，并返回那些结果为 `true` 的键值对。

## 参数

- `any_map`：映射值。

- `array<boolean>`：用于评估映射值的布尔数组。

- `lambda_func`：用于评估映射值的 Lambda 表达式。

## 返回值

返回一个数据类型与 `any_map` 相同的映射。

如果 `any_map` 为 NULL，则返回 NULL。如果 `array<boolean>` 为 null，则返回一个空映射。

如果映射值中的键或值为 NULL，则将 NULL 作为正常值处理。

Lambda 表达式必须有两个参数。第一个参数代表键。第二个参数代表值。

## 示例

### 使用 `array<boolean>`

以下示例使用 [map_from_arrays()](map_from_arrays.md) 生成映射值 `{1:"ab",3:"cdd",2:null,null:"abc"}`。然后根据 `array<boolean>` 评估每个键值对，并返回评估结果为 `true` 的键值对。

```SQL
mysql> select map_filter(col_map, array<boolean>[0,0,0,1,1]) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+----------------------------------------------------+
| map_filter(col_map, ARRAY<BOOLEAN>[0, 0, 0, 1, 1]) |
+----------------------------------------------------+
| {null:"abc"}                                       |
+----------------------------------------------------+
1 行在集合中 (0.02 秒)

mysql> select map_filter(null, array<boolean>[0,0,0,1,1]);
+-------------------------------------------------+
| map_filter(NULL, ARRAY<BOOLEAN>[0, 0, 0, 1, 1]) |
+-------------------------------------------------+
| NULL                                            |
+-------------------------------------------------+
1 行在集合中 (0.02 秒)

mysql> select map_filter(col_map, null) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+---------------------------+
| map_filter(col_map, NULL) |
+---------------------------+
| {}                        |
+---------------------------+
1 行在集合中 (0.01 秒)
```

### 使用 Lambda 表达式

以下示例使用 [map_from_arrays()](map_from_arrays.md) 生成映射值 `{1:"ab",3:"cdd",2:null,null:"abc"}`。然后根据 Lambda 表达式评估每个键值对，并返回值不为 null 的键值对。

```SQL

mysql> select map_filter((k,v) -> v is not null, col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------------+
| map_filter((k,v) -> v is not null, col_map)    |
+------------------------------------------------+
| {1:"ab",3:"cdd",null:"abc"}                    |
+------------------------------------------------+
1 行在集合中 (0.02 秒)
```