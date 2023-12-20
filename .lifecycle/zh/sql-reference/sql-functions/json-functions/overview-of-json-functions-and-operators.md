---
displayed_sidebar: English
---

# JSON 函数和运算符概述

本主题提供了 StarRocks 支持的 JSON 构造函数、查询函数、处理函数、运算符和路径表达式的概览。

## JSON 构造函数

JSON 构造函数用于创建 JSON 数据，比如 JSON 对象和 JSON 数组。

|功能|描述|示例|返回值|
|---|---|---|---|
|json_object|将一个或多个键值对转换为由键值对组成的 JSON 对象，这些键值对按字典顺序按键排序。|SELECT JSON_OBJECT(' Daniel Smith', 26, 'Lily Smith', 25 );|{“丹尼尔·史密斯”：26，“莉莉·史密斯”：25}|
|json_array|将 SQL 数组的每个元素转换为 JSON 值，并返回由这些 JSON 值组成的 JSON 数组。|SELECT JSON_ARRAY(1, 2, 3);|[1,2,3]|
|parse_json|将字符串转换为 JSON 值。|SELECT PARSE_JSON('{"a": 1}');|{"a": 1}|

## JSON 查询函数和处理函数

JSON 查询函数和处理函数用于查询和处理 JSON 数据。例如，你可以使用路径表达式定位 JSON 对象中的某个元素。

|功能|描述|示例|返回值|
|---|---|---|---|
|箭头函数|查询JSON对象中路径表达式可以定位的元素。|SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b'; |1|
|cast|在 JSON 数据类型和 SQL 数据类型之间转换数据。|SELECT CAST(1 AS JSON);|1|
|get_json_double|分析并获取 JSON 字符串中指定路径的浮点值。|SELECT get_json_double('{"k1":1.3, "k2":"2"}', "$.k1");|1.3 |
|get_json_int|分析并获取 JSON 字符串中指定路径的整数值。|SELECT get_json_int('{"k1":1, "k2":"2"}', "$.k1");|1|
|get_json_string|分析并获取 JSON 字符串中指定路径的字符串。|SELECT get_json_string('{"k1":"v1", "k2":"v2"}', "$.k1");|v1 |
|json_query|查询 JSON 对象中可通过路径表达式定位的元素的值。|SELECT JSON_QUERY('{"a": 1}', '$.a');|1|
|json_each|将 JSON 对象的顶级元素扩展为键值对。|SELECT * FROM tj_test, LATERAL JSON_EACH(j);|!json_each|
|json_exists|检查 JSON 对象是否包含可以通过路径表达式定位的元素。如果该元素存在，该函数返回 1。如果该元素不存在，该函数返回 0。|SELECT JSON_EXISTS('{"a": 1}', '$.a'); |1|
|json_keys|以 JSON 数组的形式从 JSON 对象返回顶级键，或者，如果指定了路径，则返回路径中的顶级键。|SELECT JSON_KEYS('{"a": 1, "b" : 2, "c": 3}');|["a", "b", "c"]|
|json_length|返回 JSON 文档的长度。|SELECT json_length('{"Name": "Alice"}');|1|
|json_string|将 JSON 对象转换为 JSON 字符串|SELECT json_string(parse_json('{"Name": "Alice"}'));|{"Name": "Alice"}|

## JSON 运算符

StarRocks 支持以下 JSON 比较运算符：`<`、`<=`、`>`、`>=`、`=` 和 `!=`。你可以使用这些运算符来查询 JSON 数据。但是，它不支持使用 `IN` 运算符来查询 JSON 数据。有关 JSON 运算符的更多信息，请参阅 [JSON 运算符](./json-operators.md)部分。

## JSON 路径表达式

你可以使用 JSON 路径表达式来查询 JSON 对象中的元素。JSON 路径表达式的数据类型是 STRING。在大多数情况下，它们与各种 JSON 函数一起使用，例如 JSON_QUERY。在 StarRocks 中，JSON 路径表达式并不完全符合[SQL/JSON 路径规范](https://modern-sql.com/blog/2017-06/whats-new-in-sql-2016#json-path)。有关 StarRocks 支持的 JSON 路径语法的信息，请参见下表，使用以下 JSON 对象作为示例。

```JSON
{
    "people": [{
        "name": "Daniel",
        "surname": "Smith"
    }, {
        "name": "Lily",
        "surname": "Smith",
        "active": true
    }]
}
```

|JSON 路径符号|说明|JSON 路径示例|返回值|
|---|---|---|---|
|$|表示根 JSON 对象。|'$'|{ "people": [ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname" ：史密斯，“主动”：真 } ] }|
|.|表示子 JSON 对象。|' $.people'|[ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ]|
|[]|表示一个或多个数组索引。 [n] 表示数组中的第 n 个元素。索引从0开始。StarRocks 2.5支持查询多维数组，例如[“Lucy”,“Daniel”],[“James”,“Smith”]。要查询“Lucy”元素，可以使用 $.people[0][0].|'$.people [0]'|{ "name": "Daniel", "surname": "Smith" }|
|[*]|表示数组中的所有元素。|'$.people[*].name'|["Daniel", "Lily"]|
|[start: end]|表示数组中元素的子集。子集由 [start, end] 间隔指定，其中不包括结束索引表示的元素。|'$.people[0: 1].name'|["Daniel"]|
