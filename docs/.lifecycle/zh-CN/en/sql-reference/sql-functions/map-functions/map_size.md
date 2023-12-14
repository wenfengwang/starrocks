---
displayed_sidebar: "Chinese"
---

# map_size

## 描述

返回MAP值中元素的数量。MAP是一个无序的键值对集合，例如`{"a":1, "b":2}`。一个键值对构成一个元素，例如`{"a":1, "b":2}`包含两个元素。

该函数别名为[cardinality()](cardinality.md)，从v2.5版本开始支持。

## 语法

```Haskell
INT map_size(any_map)
```

## 参数

`any_map`: 你想要获取元素数量的MAP值。

## 返回值

返回一个INT类型的值。

如果输入为NULL，则返回NULL。

如果MAP值中的键或值为NULL，则会将NULL处理为正常值。

## 示例

### 从StarRocks原生表中查询MAP数据

从v3.1版本开始，StarRocks支持在创建表时定义MAP列。该示例使用了表`test_map`，其中包含以下数据：

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

获取`col_map`列每行中元素的数量。

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

### 从数据湖中查询MAP数据

该示例使用了Hive表`hive_map`，其中包含以下数据：

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

在集群中创建[Hive catalog](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)之后，你可以使用该目录和map_size()函数获取`col_map`列每行中元素的数量。

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