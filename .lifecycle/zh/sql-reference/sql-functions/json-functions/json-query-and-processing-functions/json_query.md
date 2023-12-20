---
displayed_sidebar: English
---

# json_query

## 描述

此函数用于查询JSON对象中，通过json_path表达式可定位的元素值，并返回一个JSON值。

## 语法

```Haskell
json_query(json_object_expr, json_path)
```

## 参数

- json_object_expr：该表达式代表一个JSON对象。该对象既可以是JSON列，也可以是通过JSON构造函数（如PARSE_JSON）生成的JSON对象。

- `json_path`：该表达式代表在JSON对象中某元素的路径。此参数的值为一个字符串。关于StarRocks支持的JSON路径语法的更多信息，请参见[JSON函数和操作符概览](../overview-of-json-functions-and-operators.md)。

## 返回值

返回一个JSON值。

> 如果元素不存在，json_query函数则返回SQL的NULL值。

## 示例

示例1：查询指定JSON对象中'$.a.b'表达式能够定位到的元素值。在这个例子中，json_query函数返回的JSON值为1。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

示例2：查询指定JSON对象中'$.a.c'表达式能够定位到的元素值。在这个例子中，该元素不存在，因此json_query函数返回SQL的NULL值。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> NULL
```

示例3：查询指定JSON对象中'$.a[2]'表达式能够定位到的元素值。在这个例子中，该JSON对象为一个名为a的数组，在索引2的位置包含一个元素，其值为3。因此，JSON_QUERY函数返回的JSON值为3。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 3
```

示例4：查询指定JSON对象中'$.a[3]'表达式能够定位到的元素。在这个例子中，该JSON对象为一个名为a的数组，但不包含索引3的位置的元素。因此，json_query函数返回SQL的NULL值。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> NULL
```
