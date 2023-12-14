---
displayed_sidebar: "中文"
---

# 箭头函数

## 描述

在 JSON 对象中，通过 `json_path` 表达式可以定位的元素，并返回一个 JSON 值。箭头函数 `->` 比 [json_query](json_query.md) 函数更简洁、更易使用。

## 语法

```Haskell
json_object_expr -> json_path
```

## 参数

- `json_object_expr`：表示 JSON 对象的表达式。该对象可以是一个 JSON 列，也可以是由 PARSE_JSON 等 JSON 构造函数生成的 JSON 对象。

- `json_path`：表示 JSON 对象中元素路径的表达式。该参数的值为字符串。有关 StarRocks 支持的 JSON 路径语法的信息，请参阅[JSON 函数和操作符概述](../overview-of-json-functions-and-operators.md)。

## 返回值

返回一个 JSON 值。

> 如果该元素不存在，箭头函数将返回一个 SQL 值为 `NULL`。

## 示例

示例 1：查询指定 JSON 对象中可以通过 `'$.a.b'` 表达式定位的元素。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';

       -> 1
```

示例 2：使用嵌套的箭头函数来查询元素。将一个箭头函数嵌套在另一个箭头函数中，以基于嵌套箭头函数返回的结果来查询元素。

> 在此示例中，`json_path` 表达式中省略了根元素 $。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}')->'a'->'b';

       -> 1
```

示例 3：查询指定 JSON 对象中可以通过 `'a'` 表达式定位的元素。

> 在此示例中，`json_path` 表达式中省略了根元素 $。

```plaintext
mysql> SELECT parse_json('{"a": "b"}') -> 'a';

       -> "b"
```