---
displayed_sidebar: English
---

# json_each

## 描述

将 JSON 对象的最外层元素展开成一组键值对，这些键值对保存在两列中，并返回一个表，该表由每个元素对应一行组成。

## 语法

```Haskell
json_each(json_object_expr)
```

## 参数

`json_object_expr`：表示 JSON 对象的表达式。该对象可以是 JSON 列，也可以是由 JSON 构造函数生成的 JSON 对象，例如 PARSE_JSON。

## 返回值

返回两列：一列名为 key，另一列名为 value。key 列存储 VARCHAR 类型的值，value 列存储 JSON 类型的值。

## 使用说明

json_each 函数是一个表函数，它返回一个表。返回的表是一个由多行组成的结果集。因此，在 FROM 子句中必须使用横向连接（lateral join）将返回的表与原始表连接起来。横向连接是必须的，但是 LATERAL 关键字是可选的。json_each 函数不能在 SELECT 子句中使用。

## 示例

```plaintext
-- 使用名为 tj 的表作为示例。在 tj 表中，j 列是一个 JSON 对象。
mysql> SELECT * FROM tj;
+------+------------------+
| id   | j                |
+------+------------------+
|    1 | {"a": 1, "b": 2} |
|    3 | {"a": 3}         |
+------+------------------+

-- 通过 key 和 value 将 tj 表的 j 列展开成两列，以获得由多行组成的结果集。在此示例中，使用了 LATERAL 关键字将结果集与 tj 表连接。

mysql> SELECT * FROM tj, LATERAL json_each(j);
+------+------------------+------+-------+
| id   | j                | key  | value |
+------+------------------+------+-------+
|    1 | {"a": 1, "b": 2} | a    | 1     |
|    1 | {"a": 1, "b": 2} | b    | 2     |
|    3 | {"a": 3}         | a    | 3     |
+------+------------------+------+-------+
```