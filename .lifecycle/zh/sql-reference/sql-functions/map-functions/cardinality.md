---
displayed_sidebar: English
---

# 基数

## 描述

返回 MAP 类型值中的元素个数。MAP 是一个无序的键值对集合，例如 {"a":1, "b":2}。每个键值对构成一个元素。{"a":1, "b":2} 包含两个元素。

该功能从 v3.0 版本开始支持。它是 [map_size()](map_size.md) 函数的别名。

## 语法

```Haskell
INT cardinality(any_map)
```

## 参数

any_map：你想要获取元素个数的 MAP 类型值。

## 返回值

返回一个 INT 类型的值。

如果输入为 NULL，则返回 NULL。

如果 MAP 类型值中的键或值是 NULL，那么 NULL 将被当作一个正常的值处理。

## 示例

### 从 StarRocks 原生表中查询 MAP 数据

从 v3.1 版本开始，StarRocks 支持在创建表时定义 MAP 类型的列。本例中使用的 test_map 表包含以下数据：

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

获取 col_map 列每一行的元素个数。

```Plaintext
select cardinality(col_map) from test_map order by col_int;
+----------------------+
| cardinality(col_map) |
+----------------------+
|                    2 |
|                    1 |
|                    2 |
+----------------------+
3 rows in set (0.05 sec)
```

### 从数据湖中查询 MAP 数据

本例使用 Hive 表 hive_map，其中包含以下数据：

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

在您的集群中创建了[Hive 目录](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)后，您可以使用这个目录和`cardinality()`函数来获取`col_map`列每一行的元素个数。

```Plaintext
SELECT cardinality(col_map) FROM hive_map ORDER BY col_int;
+----------------------+
| cardinality(col_map) |
+----------------------+
|                    2 |
|                    1 |
|                    2 |
+----------------------+
3 rows in set (0.05 sec)
```
