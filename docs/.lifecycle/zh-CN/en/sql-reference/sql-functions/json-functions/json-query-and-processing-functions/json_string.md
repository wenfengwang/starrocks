---
displayed_sidebar: "Chinese"
---

# json_string

## 描述

将JSON对象转换为JSON字符串

## 语法

```SQL
json_string(json_object_expr)
```

## 参数

- `json_object_expr`: 表示JSON对象的表达式。该对象可以是一个JSON列，也可以是由PARSE_JSON等JSON构造函数生成的JSON对象。

## 返回值

返回一个VARCHAR值。

## 示例

示例1：将JSON对象转换为JSON字符串

```Plain
select json_string('{"Name": "Alice"}');
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```

示例2：将PARSE_JSON的结果转换为JSON字符串

```Plain
select json_string(parse_json('{"Name": "Alice"}'));
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```