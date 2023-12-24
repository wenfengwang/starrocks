---
displayed_sidebar: English
---

# map_values

## 描述

返回指定映射中所有值的数组。

此功能从 v2.5 版本开始支持。

## 语法

```Haskell
map_values(any_map)
```

## 参数

`any_map`：要检索值的 MAP 值。

## 返回值

返回值的格式为 `array<valueType>`。数组中的元素类型与映射中的值类型匹配。

如果输入为 NULL，则返回 NULL。如果 MAP 值中的键或值为 NULL，则将其视为正常值并包含在结果中。

## 例子

### 查询 StarRocks 原生表的 MAP 数据

从 v3.1 版本开始，StarRocks 支持在创建表时定义 MAP 列。此示例使用 `test_map` 表，其中包含以下数据：

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

获取 `col_map` 列的每一行中的所有值。

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

在群集中创建[Hive目录](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)后，您可以使用此目录和 map_values() 函数从 `col_map` 列的每一行中获取所有值。

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
