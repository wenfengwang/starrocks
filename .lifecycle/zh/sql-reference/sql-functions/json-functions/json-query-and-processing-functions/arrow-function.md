---
displayed_sidebar: English
---

# 箭头函数

## 描述

此函数用于查询JSON对象中可以通过`json_path`表达式定位的元素，并返回一个JSON值。相比[json_query](json_query.md)函数，箭头函数`->`更加简洁且易于使用。

## 语法

```Haskell
json_object_expr -> json_path
```

## 参数

- json_object_expr：代表JSON对象的表达式。这个对象可以是一个JSON列，或者是通过JSON构造函数（如PARSE_JSON）生成的JSON对象。

- `json_path`：代表JSON对象中元素路径的表达式。此参数的值为一个字符串。关于StarRocks所支持的JSON路径语法的详细信息，请参见[JSON函数和运算符概览](../overview-of-json-functions-and-operators.md)。

## 返回值

返回一个JSON值。

> 如果元素不存在，箭头函数会返回一个SQL的NULL值。

## 示例

示例1：查询指定JSON对象中可以通过'$.a.b'表达式定位的元素。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';

       -> 1
```

示例2：使用嵌套的箭头函数来查询元素。在这个例子中，一个箭头函数嵌套了另一个箭头函数，它根据被嵌套箭头函数返回的结果来查询元素。

> 在此示例中，json_path表达式省略了根元素$。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}')->'a'->'b';

       -> 1
```

示例3：查询指定JSON对象中可以通过'a'表达式定位的元素。

> 在此示例中，json_path表达式省略了根元素$。

```plaintext
mysql> SELECT parse_json('{"a": "b"}') -> 'a';

       -> "b"
```
