---
displayed_sidebar: English
---

# json_each

## 描述

将 JSON 对象的最外层元素展开为一组键值对，这些键值对保存在两列中，并返回一个表，该表由每个元素的一行组成。

## 语法

```Haskell
json_each(json_object_expr)
```

## 参数

`json_object_expr`：表示 JSON 对象的表达式。该对象可以是 JSON 列，也可以是由 PARSE_JSON 等 JSON 构造函数生成的 JSON 对象。

## 返回值

返回两列：一个名为 key，一个名为 value。key 列存储 VARCHAR 值，value 列存储 JSON 值。

## 使用说明

json_each 函数是一个返回表的表函数。返回的表是由多行组成的结果集。因此，在 FROM 子句中必须使用 lateral join 将返回的表与原始表联接。lateral join 是必需的，但 LATERAL 关键字是可选的。json_each 函数不能在 SELECT 子句中使用。

## 例子

```plaintext
-- 以名为 tj 的表为例。在 tj 表中，j 列是一个 JSON 对象。
mysql> SELECT * FROM tj;
+------+------------------+
| id   | j                |
+------+------------------+
|    1 | {"a": 1, "b": 2} |
|    3 | {"a": 3}         |
+------+------------------+

-- 将 tj 表的 j 列展开为 key 和 value 两列，以获得由多行组成的结果集。在此示例中，使用 LATERAL 关键字将结果集联接到 tj 表。

mysql> SELECT * FROM tj, LATERAL json_each(j);
+------+------------------+------+-------+
| id   | j                | key  | value |
+------+------------------+------+-------+
|    1 | {"a": 1, "b": 2} | a    | 1     |
|    1 | {"a": 1, "b": 2} | b    | 2     |
|    3 | {"a": 3}         | a    | 3     |
+------+------------------+------+-------+