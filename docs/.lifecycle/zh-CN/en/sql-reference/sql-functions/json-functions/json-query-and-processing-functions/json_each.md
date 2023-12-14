---
displayed_sidebar: "Chinese"
---

# json_each

## 描述

将JSON对象的最外层元素扩展为一组键值对，分别存储在两列中，并返回由每个元素组成的表。

## 语法

```Haskell
json_each(json_object_expr)
```

## 参数

`json_object_expr`：表示JSON对象的表达式。该对象可以是一个JSON列，或者是由PARSE_JSON等JSON构造函数生成的JSON对象。

## 返回值

返回两列：一列名为key，一列名为value。key列存储VARCHAR值，value列存储JSON值。

## 使用说明

json_each函数是一个返回表的表函数。返回的表是由多行组成的结果集。因此，在FROM子句中必须使用lateral join将返回的表连接到原始表。lateral join是强制性的，但LATERAL关键字是可选的。json_each函数不能在SELECT子句中使用。

## 示例

```plaintext
-- 以名为tj的表为例。在tj表中，j列是JSON对象。
mysql> SELECT * FROM tj;
+------+------------------+
| id   | j                |
+------+------------------+
|    1 | {"a": 1, "b": 2} |
|    3 | {"a": 3}         |
+------+------------------+

-- 将tj表的j列扩展为按key和value分为两列的结果集，获取由多行组成的结果集。在本示例中，使用LATERAL关键字将结果集连接到tj表。

mysql> SELECT * FROM tj, LATERAL json_each(j);
+------+------------------+------+-------+
| id   | j                | key  | value |
+------+------------------+------+-------+
|    1 | {"a": 1, "b": 2} | a    | 1     |
|    1 | {"a": 1, "b": 2} | b    | 2     |
|    3 | {"a": 3}         | a    | 3     |
+------+------------------+------+-------+
```