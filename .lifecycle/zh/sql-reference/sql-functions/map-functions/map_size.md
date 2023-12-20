---
displayed_sidebar: English
---

# 地图尺寸

## 描述

返回 MAP 值中的元素数量。MAP 是一个无序的键值对集合，例如，{"a":1, "b":2}。一个键值对构成一个元素，如 {"a":1, "b":2} 包含两个元素。

这个函数有一个别名是 [cardinality()](cardinality.md)。它从 v2.5 版本开始支持。

## 语法

```Haskell
INT map_size(any_map)
```

## 参数

any_map：您想要检索元素数量的 MAP 值。

## 返回值

返回一个 INT 类型的值。

如果输入是 NULL，那么返回 NULL。

如果 MAP 值中的键或值是 NULL，那么 NULL 被当作一个普通值处理。

## 示例

### 从 StarRocks 原生表中查询 MAP 数据

从 v3.1 版本开始，StarRocks 支持在创建表时定义 MAP 类型的列。本例中使用的表 test_map 包含以下数据：

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

获取 col_map 列每一行的元素数量。

```Plaintext
select map_size(col_map) from test_map order by col_int;
+-------------------+
| map_size(col_map) |
+-------------------+
|                 2 |
|                 1 |
|                 2 |
+-------------------+
3 rows in set (0.05 sec)
```

### 从数据湖中查询 MAP 数据

本例使用 Hive 表 hive_map，该表包含以下数据：

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

在您的集群中创建[Hive 目录](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)之后，您可以使用这个目录和`map_size()`函数来获取`col_map`列每一行的元素数量。

```Plaintext
select map_size(col_map) from hive_map order by col_int;
+-------------------+
| map_size(col_map) |
+-------------------+
|                 2 |
|                 1 |
|                 2 |
+-------------------+
3 rows in set (0.05 sec)
```
