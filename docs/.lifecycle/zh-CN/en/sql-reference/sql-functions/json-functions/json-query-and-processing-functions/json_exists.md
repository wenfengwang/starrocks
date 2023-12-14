---
displayed_sidebar: "English"
---

# json_exists

## 描述

检查JSON对象是否包含可以通过`json_path`表达式定位的元素。如果该元素存在，则JSON_EXISTS函数返回`1`。否则，JSON_EXISTS函数返回`0`。

## 语法

```Haskell
json_exists(json_object_expr, json_path)
```

## 参数

- `json_object_expr`：表示JSON对象的表达式。该对象可以是JSON列，或者是由PARSE_JSON等JSON构造函数生成的JSON对象。

- `json_path`：表示JSON对象中元素路径的表达式。该参数的值为字符串。有关StarRocks支持的JSON路径语法的更多信息，请参见[JSON函数和运算符概述](../overview-of-json-functions-and-operators.md)。

## 返回值

返回布尔值。

## 示例

示例1：检查指定的JSON对象是否包含可以通过`'$.a.b'`表达式定位的元素。在此示例中，JSON对象中存在该元素。因此，json_exists函数返回`1`。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

示例2：检查指定的JSON对象是否包含可以通过`'$.a.c'`表达式定位的元素。在此示例中，JSON对象中不存在该元素。因此，json_exists函数返回`0`。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> 0
```

示例3：检查指定的JSON对象是否包含可以通过`'$.a[2]'`表达式定位的元素。在此示例中，JSON对象（名为a的数组）中包含索引为2的元素。因此，json_exists函数返回`1`。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 1
```

示例4：检查指定的JSON对象是否包含可以通过`'$.a[3]'`表达式定位的元素。在此示例中，JSON对象（名为a的数组）中不包含索引为3的元素。因此，json_exists函数返回`0`。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> 0
```