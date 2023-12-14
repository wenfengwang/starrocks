---
displayed_sidebar: "Chinese"
---

# JSON函数和运算符概述

本主题提供了JSON构造函数、查询函数和处理函数、运算符以及路径表达式的概述，这些函数、运算符和路径表达式是StarRocks支持的。

## JSON构造函数

JSON构造函数用于构造JSON数据，例如JSON对象和JSON数组。

| 函数名                                                      | 描述                                   | 示例                                               | 返回值                            |
| ------------------------------------------------------------ | -------------------------------------- | ---------------------------------------------------- | ----------------------------------- |
| [json_object](./json-constructor-functions/json_object.md) | 将一个或多个键值对转换为按字典序排列的JSON对象。 | `SELECT JSON_OBJECT(' Daniel Smith', 26, 'Lily Smith', 25);` | `{"Daniel Smith": 26, "Lily Smith": 25}` |
| [json_array](./json-constructor-functions/json_array.md) | 将SQL数组的每个元素转换为JSON值，并返回由这些JSON值组成的JSON数组。 | `SELECT JSON_ARRAY(1, 2, 3);`                        | `[1,2,3]`                           |
| [parse_json](./json-constructor-functions/parse_json.md)  | 将字符串转换为JSON值。             | `SELECT PARSE_JSON('{"a": 1}');`                     | `{"a": 1}`                         |

## JSON查询函数和处理函数

JSON查询函数和处理函数用于查询和处理JSON数据。例如，您可以使用路径表达式定位JSON对象中的元素。

| 函数名                                                      | 描述                                   | 示例                                        | 返回值                                   |
| ------------------------------------------------------------ | -------------------------------------- | ------------------------------------------- | ------------------------------------------ |
| [arrow function](./json-query-and-processing-functions/arrow-function.md) | 通过路径表达式查询JSON对象中的元素。 | `SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';`                          | `1`                                      |
| [cast](./json-query-and-processing-functions/cast.md) | 在JSON数据类型和SQL数据类型之间进行数据转换。 | `SELECT CAST(1 AS JSON);`                 | `1`                                      |
| [get_json_double](./json-query-and-processing-functions/get_json_double.md) | 从JSON字符串中的指定路径分析并获取浮点数值。 | `SELECT get_json_double('{"k1":1.3, "k2":"2"}', "$.k1");` | `1.3`                                |
| [get_json_int](./json-query-and-processing-functions/get_json_int.md) | 从JSON字符串的指定路径分析并获取整数值。 | `SELECT get_json_int('{"k1":1, "k2":"2"}', "$.k1");` | `1`                                   |
| [get_json_string](./json-query-and-processing-functions/get_json_string.md) | 从JSON字符串的指定路径中分析并获取字符串值。 | `SELECT get_json_string('{"k1":"v1", "k2":"v2"}', "$.k1");` | `v1`                                |
| [json_query](./json-query-and-processing-functions/json_query.md) | 通过路径表达式查询JSON对象中的元素值。 | `SELECT JSON_QUERY('{"a": 1}', '$.a');`                | `1`                                 |
| [json_each](./json-query-and-processing-functions/json_each.md) | 将JSON对象的顶级元素展开为键值对。 | `SELECT * FROM tj_test, LATERAL JSON_EACH(j);` | `!`[json_each](../../../assets/json_each.png) |
| [json_exists](./json-query-and-processing-functions/json_exists.md) | 检查JSON对象是否包含通过路径表达式定位的元素。如果存在，则返回1。如果不存在，则返回0。 | `SELECT JSON_EXISTS('{"a": 1}', '$.a');`                       | `1`                                   |
| [json_keys](./json-query-and-processing-functions/json_keys.md) | 将JSON对象的顶级键作为JSON数组返回，如果指定了路径，则返回该路径的顶级键。   | `SELECT JSON_KEYS('{"a": 1, "b": 2, "c": 3}');` |  `["a", "b", "c"]`                   |
| [json_length](./json-query-and-processing-functions/json_length.md) | 返回JSON文档的长度。 | `SELECT json_length('{"Name": "Alice"}');` |  `1`                                   |
| [json_string](./json-query-and-processing-functions/json_string.md) | 将JSON对象转换为JSON字符串。     | `SELECT json_string(parse_json('{"Name": "Alice"}'));` | `{"Name": "Alice"}`                    |

## JSON运算符

StarRocks支持以下JSON比较运算符：`<`、`<=`、`>`、`>=`、`=`和`!=`。您可以使用这些运算符来查询JSON数据。但是，不允许使用`IN`来查询JSON数据。有关JSON运算符的详细信息，请参见 [JSON运算符](./json-operators.md)。

## JSON路径表达式

您可以使用JSON路径表达式查询JSON对象中的元素。JSON路径表达式是STRING数据类型。在大多数情况下，它们与各种JSON函数一起使用，如JSON_QUERY。在StarRocks中，JSON路径表达式不完全符合[SQL/JSON路径规范](https://modern-sql.com/blog/2017-06/whats-new-in-sql-2016#json-path)。有关StarRocks支持的JSON路径语法的信息，请参见以下表格，表格中使用以下JSON对象作为示例。

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

| JSON路径符号 | 描述                                   | JSON路径示例 | 返回值                              |
| ------------- | -------------------------------------- | ----------- | ------------------------------------- |
| `$`           | 表示根JSON对象。                     | `'$'`          | `{ "people": [ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ] }`|
|`.`           | 表示子JSON对象。                     |`' $.people'` |`[ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ]`|
|`[]`          | 表示一个或多个数组索引。`[n]`表示数组中的第n个元素。索引从0开始。<br />StarRocks 2.5支持查询多维数组，例如 `["Lucy", "Daniel"], ["James", "Smith"]`。要查询 "Lucy" 元素，可以使用 `$.people[0][0]`。   | `'$.people [0]'` | `{ "name": "Daniel", "surname": "Smith" }` |
| `[*]`        | 表示数组中的所有元素。                | `'$.people[*].name'` | `["Daniel", "Lily"]`                  |
| `[start: end]` | 表示数组中的一部分元素。子集由`[start, end]`间隔指定，不包括由结束索引表示的元素。| `'$.people[0: 1].name'` | `["Daniel"]`                                |