---
displayed_sidebar: English
---

# json_string

## 描述

将 JSON 对象转换为 JSON 字符串

## 语法

```SQL
json_string(json_object_expr)
```

## 参数

- `json_object_expr`：表示 JSON 对象的表达式。该对象可以是 JSON 列，也可以是由 JSON 构造函数（例如 PARSE_JSON）生成的 JSON 对象。

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

示例 1：将 JSON 对象转换为 JSON 字符串

```Plain
select json_string('{"Name": "Alice"}');
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```

示例 2：将 PARSE_JSON 的结果转换为 JSON 字符串

```Plain
select json_string(parse_json('{"Name": "Alice"}'));
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```