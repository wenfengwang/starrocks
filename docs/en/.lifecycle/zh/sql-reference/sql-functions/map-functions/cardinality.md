---
displayed_sidebar: English
---

# cardinality

## 描述

返回 MAP 值中的元素数量。MAP 是一个无序的键值对集合，例如 `{"a":1, "b":2}`。一个键值对构成一个元素。`{"a":1, "b":2}` 包含两个元素。

该函数从 v3.0 版本开始支持。它是 [map_size()](map_size.md) 的别名。

## 语法

```Haskell
INT cardinality(any_map)
```

## 参数

`any_map`：你想要获取元素数量的 MAP 值。

## 返回值

返回一个 INT 类型的值。

如果输入为 NULL，则返回 NULL。

如果 MAP 值中的键或值为 NULL，NULL 将被当作一个正常的值处理。

## 示例

### 从 StarRocks 原生表中查询 MAP 数据

从 v3.1 版本开始，StarRocks 支持在创建表时定义 MAP 类型的列。此示例使用 `test_map` 表，包含以下数据：

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

获取 `col_map` 列每行的元素数量。

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

### 从数据湖查询 MAP 数据

此示例使用 Hive 表 `hive_map`，包含以下数据：

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

在集群中创建 [Hive 目录](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog) 后，你可以使用这个目录和 `cardinality()` 函数来获取 `col_map` 列每行的元素数量。

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