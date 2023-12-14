---
displayed_sidebar: "Chinese"
---

# 基数

## 描述

返回MAP值中元素的数量。MAP是无序的键值对集合，例如 `{"a":1, "b":2}`。一个键值对构成一个元素。 `{"a":1, "b":2}` 包含两个元素。

从v3.0版本开始支持此函数。这是[map_size()](map_size.md)的别名。

## 语法

```Haskell
INT 基数(any_map)
```

## 参数

`any_map`: 您想要检索元素数量的MAP值。

## 返回值

返回一个INT值。

如果输入为NULL，则返回NULL。

如果MAP值中的键或值为NULL，则将NULL视为正常值进行处理。

## 示例

### 从StarRocks原生表中查询MAP数据

从v3.1版本开始，StarRocks支持在创建表时定义MAP列。此示例使用表`test_map`，其中包含以下数据：

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
select 基数(col_map) from test_map order by col_int;
+-------------------+
| 基数(col_map)      |
+-------------------+
|                 2 |
|                 1 |
|                 2 |
+-------------------+
3 rows in set (0.05 sec)
```

### 从数据湖中查询MAP数据

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

在集群中创建[Hive catalog](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)后，您可以使用此catalog和`基数()`函数获取`col_map`列每行中元素的数量。

```Plaintext
SELECT 基数(col_map) FROM hive_map ORDER BY col_int;
+-------------------+
| 基数(col_map)      |
+-------------------+
|                 2 |
|                 1 |
|                 2 |
+-------------------+
3 rows in set (0.05 sec)
```