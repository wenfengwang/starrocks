---
displayed_sidebar: English
---

# json_string

## 描述

将 JSON 对象转换成 JSON 字符串

## 语法

```SQL
json_string(json_object_expr)
```

## 参数

- json_object_expr：代表 JSON 对象的表达式。该对象可以是一个 JSON 列，或者是由 JSON 构造函数生成的 JSON 对象，例如 PARSE_JSON。

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

示例 1：把 JSON 对象转换成 JSON 字符串

```Plain
select json_string('{"Name": "Alice"}');
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```

示例 1：将 PARSE_JSON 函数的结果转换为 JSON 字符串

```Plain
select json_string(parse_json('{"Name": "Alice"}'));
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```
