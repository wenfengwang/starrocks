---
displayed_sidebar: English
---

# 地图值函数

## 描述

该函数返回指定映射中所有值组成的数组。

此功能自v2.5版本起提供支持。

## 语法

```Haskell
map_values(any_map)
```

## 参数

any_map：你想要从中提取值的MAP类型的值。

## 返回值

返回值格式为 array<valueType>。数组中的元素类型与映射中的值类型一致。

如果输入为NULL，则返回NULL。如果MAP值中的键或值为NULL，那么NULL会被当做普通值处理，并包含在结果中。

## 示例

### 从StarRocks原生表中查询MAP数据

从v3.1版本起，StarRocks开始支持在创建表时定义MAP类型的列。本例中使用名为test_map的表，该表包含以下数据：

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

提取col_map列每一行中的所有值。

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

### 从数据湖中查询MAP数据

本例使用名为hive_map的Hive表，该表包含以下数据：

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

在集群中创建[Hive目录](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)后，你可以利用该目录和`map_values()`函数来提取`col_map`列每一行中的所有值。

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
