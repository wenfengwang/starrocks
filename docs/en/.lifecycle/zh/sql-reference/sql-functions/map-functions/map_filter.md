---
displayed_sidebar: English
---

# map_filter

## 描述

通过将布尔数组或 Lambda 表达式应用于映射中的每个键值对来筛选键值对。返回计算结果为`true`的对。

此功能从v3.1开始受支持。

## 语法

```Haskell
MAP map_filter(any_map, array<boolean>)
MAP map_filter(lambda_func, any_map)
```

- `map_filter(any_map, array<boolean>)`

   逐个计算`any_map`中的键值对，根据`array<boolean>`进行评估，并返回计算结果为`true`的键值对。

- `map_filter(lambda_func, any_map)`

  逐个将`lambda_func`应用于`any_map`中的键值对，并返回计算结果为`true`的键值对。

## 参数

- `any_map`：映射值。

- `array<boolean>`：用于评估映射值的布尔数组。

- `lambda_func`：用于评估映射值的Lambda表达式。

## 返回值

返回数据类型与`any_map`相同的映射。

如果`any_map`为NULL，则返回NULL。如果`array<boolean>`为null，则返回空映射。

如果映射值中的键或值为NULL，则将NULL视为正常值处理。

Lambda表达式必须具有两个参数。第一个参数表示键，第二个参数表示值。

## 例子

### 使用`array<boolean>`

以下示例使用[map_from_arrays()](map_from_arrays.md)生成映射值`{1:"ab",3:"cdd",2:null,null:"abc"}`。然后，根据每个键值对进行评估`array<boolean>`，并返回计算结果为`true`的键值对。

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

以下示例使用map_from_arrays()生成映射值`{1:"ab",3:"cdd",2:null,null:"abc"}`。然后，根据Lambda表达式评估每个键值对，并返回值不为null的键值对。

```SQL

mysql> select map_filter((k,v) -> v is not null,col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------------+
| map_filter((k,v) -> v is not null, col_map)    |
+------------------------------------------------+
| {1:"ab",3:"cdd",null:'abc'}                        |
+------------------------------------------------+
1 row in set (0.02 sec)
```
