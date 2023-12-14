---
displayed_sidebar: "Chinese"
---

# json_query

## 描述

在 JSON 对象中通过 `json_path` 表达式定位元素的值，并返回一个 JSON 值。

## 语法

```Haskell
json_query(json_object_expr, json_path)
```

## 参数

- `json_object_expr`：表示 JSON 对象的表达式。该对象可以是一个 JSON 列，也可以是由 JSON 构造函数（例如 PARSE_JSON）生成的 JSON 对象。

- `json_path`：表示 JSON 对象中元素路径的表达式。该参数的值为一个字符串。有关 StarRocks 支持的 JSON 路径语法的信息，请参见[JSON 函数和操作符概述](../overview-of-json-functions-and-operators.md)。

## 返回值

返回一个 JSON 值。

> 如果元素不存在，则 `json_query` 函数返回一个 SQL 值为 `NULL`。

## 示例

示例 1：查询指定 JSON 对象中可以通过 `'$.a.b'` 表达式定位的元素的值。在此示例中，`json_query` 函数返回一个 JSON 值 `1`。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

示例 2：查询指定 JSON 对象中可以通过 `'$.a.c'` 表达式定位的元素的值。在此示例中，该元素不存在。因此，`json_query` 函数返回一个 SQL 值为 `NULL`。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> NULL
```

示例 3：查询指定 JSON 对象中可以通过 `'$.a[2]'` 表达式定位的元素的值。在此示例中，JSON 对象是一个名为 a 的数组，包含索引为 2 的元素，其值为 3。因此，`JSON_QUERY` 函数返回一个 JSON 值 `3`。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 3
```

示例 4：查询指定 JSON 对象中可以通过 `'$.a[3]'` 表达式定位的元素的值。在此示例中，JSON 对象是一个名为 a 的数组，不包含索引为 3 的元素。因此，`json_query` 函数返回一个 SQL 值为 `NULL`。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> NULL
```