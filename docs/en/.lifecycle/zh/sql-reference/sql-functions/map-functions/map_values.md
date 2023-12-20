---
displayed_sidebar: English
---

# map_values

## 描述

返回指定映射中所有值的数组。

该函数从v2.5版本开始支持。

## 语法

```Haskell
map_values(any_map)
```

## 参数

`any_map`：要从中检索值的MAP值。

## 返回值

返回值的格式为`array<valueType>`。数组中的元素类型与映射中的值类型相匹配。

如果输入为NULL，则返回NULL。如果MAP值中的键或值为NULL，NULL将作为正常值处理并包含在结果中。

## 示例

### 从 StarRocks 本地表查询 MAP 数据

从v3.1版本开始，StarRocks 支持在创建表时定义MAP列。本示例使用表`test_map`，其中包含以下数据：

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
3行在集合中 (0.05秒)
```

获取`col_map`列的每一行的所有值。

```SQL
select map_values(col_map) from test_map order by col_int;
+---------------------+
| map_values(col_map) |
+---------------------+
| [1,2]               |
| [3]                 |
| [4,5]               |
+---------------------+
3行在集合中 (0.04秒)
```

### 从数据湖查询 MAP 数据

本示例使用Hive表`hive_map`，其中包含以下数据：

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

在您的集群中创建[Hive Catalog](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)后，您可以使用该Catalog和`map_values()`函数来获取`col_map`列每一行的所有值。

```SQL
select map_values(col_map) from hive_map order by col_int;
+---------------------+
| map_values(col_map) |
+---------------------+
| [1,2]               |
| [3]                 |
| [4,5]               |
+---------------------+
3行在集合中 (0.04秒)
```