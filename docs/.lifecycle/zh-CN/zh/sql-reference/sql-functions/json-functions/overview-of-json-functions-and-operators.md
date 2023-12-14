---
displayed_sidebar: "Chinese"
---

# JSON 函数概述

StarRocks支持以下JSON构造函数、JSON查询和处理函数、JSON运算符以及查询JSON对象的JSON Path。

## JSON 构造函数

JSON构造函数可以构造JSON类型的数据。例如JSON类型的对象、JSON类型的数组等。

| 函数名称                                                     | 功能                                 | 示例                                                      | 返回结果                               |
| ------------------------------------------------------------ | ------------------------------------ | --------------------------------------------------------- | -------------------------------------- |
| [json_object](../../sql-functions/json-functions/json-constructor-functions/json_object.md) | 构造JSON类型的对象。                      | `SELECT JSON_OBJECT(' Daniel Smith', 26, 'Lily Smith', 25);` | `{"Daniel Smith": 26, "Lily Smith": 25}` |
| [json_array](../../sql-functions/json-functions/json-constructor-functions/json_array.md)   | 构造JSON类型的数组。                    | `SELECT JSON_ARRAY(1, 2, 3);`                                | `[1,2,3]`                                |
| [parse_json](../../sql-functions/json-functions/json-constructor-functions/parse_json.md)   | 从字符串解析并构造出JSON类型的数据。    | `SELECT PARSE_JSON('{"a": 1}');`                             | `{"a": 1}`                               |

## JSON查询和处理函数

JSON查询和处理函数可以查询和处理JSON类型的数据。例如查询JSON对象中指定路径下的值。

| 函数名称                                                     | 功能                                 | 示例                                                      | 返回结果                              |
| ------------------------------------------------------------ | ------------------------------------ | --------------------------------------------------------- | -------------------------------------- |
| [箭头函数](../../sql-functions/json-functions/json-query-and-processing-functions/arrow-function.md) | 查询JSON对象中指定路径下的值。                         | `SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';`         | `1`                                                            |
| [JSON类型转换](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md)| 将实现JSON类型的数据与SQL类型间的互相转换。      | `SELECT CAST(1 AS JSON);` |  `1` |
| [get_json_double](../../sql-functions/json-functions/json-query-and-processing-functions/get_json_double.md)| 解析并获取json_str内json_path的浮点型内容。      | `SELECT get_json_double('{"k1":1.3, "k2":"2"}', "$.k1");`  |  `1.3` |
| [get_json_int](../../sql-functions/json-functions/json-query-and-processing-functions/get_json_int.md)| 解析并获取json_str内json_path的整型内容。      | `SELECT get_json_int('{"k1":1, "k2":"2"}', "$.k1");` |  `1` |
| [get_json_string](../../sql-functions/json-functions/json-query-and-processing-functions/get_json_string.md)| 解析并获取json_str内json_path指定的字符串。该函数别名为get_json_object。      | `SELECT get_json_string('{"k1":"v1", "k2":"v2"}', "$.k1");`| `v1` |
| [json_each](../../sql-functions/json-functions/json-query-and-processing-functions/json_each.md)   | 将最外层的JSON对象展开为键值对。      | `SELECT * FROM tj_test, LATERAL JSON_EACH(j);` | ![json_each](../../../assets/json_each.png) |
| [json_exists](../../sql-functions/json-functions/json-query-and-processing-functions/json_exists.md)| 查询JSON对象中是否存在某个值。如果存在，则返回1；如果不存在，则返回0。 | `SELECT JSON_EXISTS('{"a": 1}', '$.a');`            | `1`                              |
| [json_keys](../../sql-functions/json-functions/json-query-and-processing-functions/json_keys.md) | 返回JSON对象中所有最上层成员(key)组成的数组。                     | `SELECT JSON_KEYS('{"a": 1, "b": 2, "c": 3}');`          | `["a", "b", "c"]`                             |
| [json_length](../../sql-functions/json-functions/json-query-and-processing-functions/json_length.md) | 返回JSON字符串的长度。    | `SELECT json_length('{"Name": "Alice"}');`    | `1`                       |
| [json_query](../../sql-functions/json-functions/json-query-and-processing-functions/json_query.md) | 查询JSON对象中指定路径下的值。                             | `SELECT JSON_QUERY('{"a": 1}', '$.a');`                    | `1`                                                            |
| [json_string](../../sql-functions/json-functions/json-query-and-processing-functions/json_string.md)   | 将JSON对象转化为JSON字符串。      | `SELECT json_string(parse_json('{"Name": "Alice"}'));` | `{"Name": "Alice"}`  |

## JSON运算符

StarRocks支持使用`<`，`<=`，`>`，`>=`，`=`，`!=`运算符查询JSON数据，不支持使用IN运算符。JSON运算符的更多说明，请参见[JSON运算符](../../sql-functions/json-functions/json-operators.md)。

## JSON Path

您可以使用JSON Path路径表达式，查询JSON类型的对象中指定路径的值。JSON Path为字符串类型，一般结合多种JSON函数使用（例如JSON_QUERY）。目前StarRocks中JSON Path没有完全遵循[SQL/JSONPath标准](https://modern-sql.com/blog/2017-06/whats-new-in-sql-2016#json-path)。StarRocks中JSON Path语法说明，参见下表（以如下JSON object为例）。

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

| JSON Path的符号   | 说明                                                         | JSON Path示例     | 查询上述JSON对象的值                           |
| ---------------- | ------------------------------------------------------------ | ----------------  | -------------------------------------------- |
| $               | 表示根节点的对象。                                           | '$'               | `{ "people": [ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ] }` |
| .               | 表示子节点。                                                 | '$.people'         | `[ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ]` |
| []              | 表示一个或多个数组下标。[n]表示选择数组中第n个元素，从0开始计数。<br />从2.5版本开始支持查询多维数组，例如["Lucy", "Daniel"],["James", "Smith"]。如果要查询到"Lucy"这个元素，可以使用路径`$.people[0][0]`。| '$.people[0]'     | `{ "name": "Daniel", "surname": "Smith"}`       |
| [*]             | 表示数组中的全部元素。                                       | '$.people[*].name' | ["Daniel", "Lily"]                            |
| [start:end]     | 表示数组片段，区间为[start, end)，不包含end代表的元素。      | '$.people[0:1].name' | ["Daniel"]                                   |