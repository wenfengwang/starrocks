---
displayed_sidebar: English
---

# 基数

## 描述

返回 MAP 值中的元素数。MAP 是键值对的无序集合，例如 `{"a":1, "b":2}`。一个键值对构成一个元素。`{"a":1, "b":2}` 包含两个元素。

此功能从 v3.0 版本开始支持。它是 [map_size() 的别名](map_size.md)。

## 语法

```Haskell
INT cardinality(any_map)
```

## 参数

`any_map`：要从中检索元素数的 MAP 值。

## 返回值

返回 INT 类型的值。

如果输入为 NULL，则返回 NULL。

如果 MAP 值中的键或值为 NULL，则将 NULL 作为正常值处理。

## 例子

### 查询 StarRocks 原生表的 MAP 数据

从 v3.1 版本开始，StarRocks 支持在创建表时定义 MAP 列。此示例使用了 `test_map` 表，其中包含以下数据：

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

获取 `col_map` 列中每行的元素数。

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

此示例使用 Hive 表 `hive_map`，其中包含以下数据：

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

在群集中创建了 [Hive 目录](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog) 后，您可以使用该目录和 cardinality() 函数来获取 `col_map` 列中每行的元素数。

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