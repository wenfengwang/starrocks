---
displayed_sidebar: English
---

# 映射键

## 描述

返回指定映射中所有键组成的数组。

该函数从v2.5版本起提供支持。

## 语法

```Haskell
map_keys(any_map)
```

## 参数

any_map：您想要从中提取键的MAP值。

## 返回值

返回值格式为array<keyType>。数组中的元素类型与映射中的键类型相匹配。

如果输入为NULL，则返回NULL。如果MAP值中的键或值为NULL，那么NULL将作为正常值处理并包含在结果中。

## 示例

### 从StarRocks原生表中查询MAP数据

从v3.1版本开始，StarRocks支持在创建表时定义MAP类型的列。本例中使用名为test_map的表，包含以下数据：

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

从col_map列的每行中获取所有键。

```Plaintext
select map_keys(col_map) from test_map order by col_int;
+-------------------+
| map_keys(col_map) |
+-------------------+
| ["a","b"]         |
| ["c"]             |
| ["d","e"]         |
+-------------------+
3 rows in set (0.05 sec)
```

### 从数据湖中查询MAP数据

本例使用Hive表hive_map，包含以下数据：

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

在您的集群中创建[Hive目录](../../../data_source/catalog/hive_catalog.md#create-a-hive-catalog)之后，您可以使用该目录和`map_keys()`函数来获取`col_map`列每行中的所有键。

```Plaintext
select map_keys(col_map) from hive_map order by col_int;
+-------------------+
| map_keys(col_map) |
+-------------------+
| ["a","b"]         |
| ["c"]             |
| ["d","e"]         |
+-------------------+
3 rows in set (0.05 sec)
```
