---
displayed_sidebar: English
---

# json_exists

## 描述

检查一个 JSON 对象是否含有可以通过 json_path 表达式定位的元素。如果该元素存在，JSON_EXISTS 函数将返回 1；若不存在，函数则返回 0。

## 语法

```Haskell
json_exists(json_object_expr, json_path)
```

## 参数

- json_object_expr：代表 JSON 对象的表达式。该对象可以是一个 JSON 列，或者是由 JSON 构造函数生成的 JSON 对象，例如 PARSE_JSON。

- `json_path`：代表 JSON 对象中某个元素路径的表达式。此参数的值为一个字符串。欲了解 StarRocks 支持的 JSON 路径语法的详细信息，请参见[JSON 函数和运算符概览](../overview-of-json-functions-and-operators.md)。

## 返回值

返回 BOOLEAN 类型的值。

## 示例

示例 1：检查指定的 JSON 对象是否包含可以通过 '$.a.b' 表达式定位的元素。在这个例子中，该元素存在于 JSON 对象内，因此 json_exists 函数返回 1。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

示例 2：检查指定的 JSON 对象是否包含可以通过 '$.a.c' 表达式定位的元素。在这个例子中，该元素在 JSON 对象中不存在，因此 json_exists 函数返回 0。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> 0
```

示例 3：检查指定的 JSON 对象是否包含可以通过 '$.a[2]' 表达式定位的元素。在这个例子中，JSON 对象是一个名为 a 的数组，它包含了索引为 2 的元素，因此 json_exists 函数返回 1。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 1
```

示例 4：检查指定的 JSON 对象是否包含可以通过 '$.a[3]' 表达式定位的元素。在这个例子中，名为 a 的数组类型的 JSON 对象并不包含索引为 3 的元素，因此 json_exists 函数返回 0。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> 0
```
