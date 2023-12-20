---
displayed_sidebar: English
---

# json_each

## 描述

此函数将 JSON 对象的最外层元素展开为两列中的键值对集合，并返回一个表格，表中的每一行对应一个元素。

## 语法

```Haskell
json_each(json_object_expr)
```

## 参数

json_object_expr：代表 JSON 对象的表达式。此对象可以是一个 JSON 列，或者是通过 JSON 构造函数（例如 PARSE_JSON）产生的 JSON 对象。

## 返回值

返回两列：一列名为“键”（key），另一列名为“值”（value）。键列存储的是 VARCHAR 类型的值，值列存储的则是 JSON 类型的值。

## 使用须知

json_each 函数是一个表函数，用于返回一个表。返回的表是一个包含多行的结果集。因此，在 FROM 子句中必须使用横向连接（lateral join）来将返回的表与原始表连接。虽然横向连接是必须的，但是使用 LATERAL 关键词则是可选的。json_each 函数不能在 SELECT 子句中使用。

## 示例

```plaintext
-- A table named tj is used as an example. In the tj table, the j column is a JSON object.
mysql> SELECT * FROM tj;
+------+------------------+
| id   | j                |
+------+------------------+
|    1 | {"a": 1, "b": 2} |
|    3 | {"a": 3}         |
+------+------------------+

-- Expand the j column of the tj table into two columns by key and value to obtain a result set that consists of multiple rows. In this example, the LATERAL keyword is used to join the result set to the tj table.

mysql> SELECT * FROM tj, LATERAL json_each(j);
+------+------------------+------+-------+
| id   | j                | key  | value |
+------+------------------+------+-------+
|    1 | {"a": 1, "b": 2} | a    | 1     |
|    1 | {"a": 1, "b": 2} | b    | 2     |
|    3 | {"a": 3}         | a    | 3     |
+------+------------------+------+-------+
```
