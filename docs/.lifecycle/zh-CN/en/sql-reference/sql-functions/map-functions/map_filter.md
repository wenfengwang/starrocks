---
displayed_sidebar: "Chinese"
---

# map_filter

## 描述

通过将布尔数组或[lambda表达式](../Lambda_expression.md)应用于每个键-值对来过滤映射中的键-值对。返回结果为`true`的键-值对。

此函数从v3.1版本开始受支持。

## 语法

```Haskell
MAP map_filter(any_map, array<boolean>)
MAP map_filter(lambda_func, any_map)
```

- `map_filter(any_map, array<boolean>)`

  逐个对`any_map`中的键-值对与`array<boolean>`进行评估，并返回评估结果为`true`的键-值对。

- `map_filter(lambda_func, any_map)`

  逐个将`lambda_func`应用于`any_map`中的键-值对，并返回评估结果为`true`的键-值对。

## 参数

- `any_map`: 映射值。

- `array<boolean>`: 用于评估映射值的布尔数组。

- `lambda_func`: 用于评估映射值的lambda表达式。

## 返回值

返回一个数据类型与`any_map`相同的映射。

如果`any_map`为NULL，则返回NULL。如果`array<boolean>`为null，则返回一个空映射。

如果映射值中的键或值为NULL，则将NULL处理为正常值。

Lambda表达式必须有两个参数。第一个参数表示键，第二个参数表示值。

## 示例

### 使用`array<boolean>`

以下示例使用[map_from_arrays()](map_from_arrays.md)生成映射值`{1:"ab",3:"cdd",2:null,null:"abc"}`。然后对每个键-值对与`array<boolean>`进行评估，并返回评估结果为`true`的键-值对。

```SQL
mysql> select map_filter(col_map, array<boolean>[0,0,0,1,1]) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+----------------------------------------------------+
| map_filter(col_map, ARRAY<BOOLEAN>[0, 0, 0, 1, 1]) |
+----------------------------------------------------+
| {null:"abc"}                                       |
+----------------------------------------------------+
1 row in set (0.02 sec)

mysql> select map_filter(null, array<boolean>[0,0,0,1,1]);
+-------------------------------------------------+
| map_filter(NULL, ARRAY<BOOLEAN>[0, 0, 0, 1, 1]) |
+-------------------------------------------------+
| NULL                                            |
+-------------------------------------------------+
1 row in set (0.02 sec)

mysql> select map_filter(col_map, null) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+---------------------------+
| map_filter(col_map, NULL) |
+---------------------------+
| {}                        |
+---------------------------+
1 row in set (0.01 sec)
```

### 使用Lambda表达式

以下示例使用map_from_arrays()生成映射值`{1:"ab",3:"cdd",2:null,null:"abc"}`。然后对每个键-值对与Lambda表达式进行评估，并返回值不为null的键-值对。

```SQL

mysql> select map_filter((k,v) -> v is not null,col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------------+
| map_filter((k,v) -> v is not null, col_map)    |
+------------------------------------------------+
| {1:"ab",3:"cdd",null:'abc'}                        |
+------------------------------------------------+
1 row in set (0.02 sec)
```