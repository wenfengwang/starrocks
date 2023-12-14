---
displayed_sidebar: "Chinese"
---

# map_values

## 描述

返回指定地图中的所有值数组。

此函数从v2.5版开始支持。

## 语法

```Haskell
map_values(any_map)
```

## 参数

`any_map`：要检索值的地图值。

## 返回值

返回值的格式为`array<valueType>`。数组中的元素类型与地图中的值类型匹配。

如果输入为NULL，则返回NULL。如果地图值中的键或值为NULL，则将NULL作为普通值处理并包含在结果中。

## 示例

### 从StarRocks原生表查询MAP数据

从v3.1开始，StarRocks支持在创建表时定义MAP列。此示例使用名为`test_map`的表，其中包含以下数据：

```Plain
CREATE TABLE test_map(
    col_int INT,
    col_map MAP<VARCHAR(50),INT>
  )
DUPLICATE KEY(col_int);

INSERT INTO test_map VALUES
(1,map{"a":1,"b":2}),
(2,map{"c":3}),
(3,map{"d":4,"e":5});

SELECT * FROM test_map ORDER BY col_int;
+---------+---------------+
| col_int | col_map       |
+---------+---------------+
|       1 | {"a":1,"b":2} |
|       2 | {"c":3}       |
|       3 | {"d":4,"e":5} |
+---------+---------------+
3 rows in set (0.05 sec)
```

获取`col_map`列的每行的所有值。

```SQL
select map_values(col_map) from test_map order by col_int;
+---------------------+
| map_values(col_map) |
+---------------------+
| [1,2]               |
| [3]                 |
| [4,5]               |
+---------------------+
3 rows in set (0.04 sec)
```

### 从数据湖查询MAP数据

此示例使用Hive表`hive_map`，其中包含以下数据：

```Plaintext
SELECT * FROM hive_map ORDER BY col_int;
+---------+---------------+
| col_int | col_map       |
+---------+---------------+
|       1 | {"a":1,"b":2} |
|       2 | {"c":3}       |
|       3 | {"d":4,"e":5} |
+---------+---------------+
```

在集群中创建[Hive catalog](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)后，可以使用此catalog和`map_values()`函数从`col_map`列的每行中获取所有值。

```SQL
select map_values(col_map) from hive_map order by col_int;
+---------------------+
| map_values(col_map) |
+---------------------+
| [1,2]               |
| [3]                 |
| [4,5]               |
+---------------------+
3 rows in set (0.04 sec)
```