---
displayed_sidebar: English
---

# JSON 函数和运算符概述

本主题提供了 StarRocks 支持的 JSON 构造函数、查询函数和处理函数、运算符以及路径表达式的概述。

## JSON 构造函数

JSON 构造函数用于构造 JSON 数据，例如 JSON 对象和 JSON 数组。

| 函数                                                     | 描述                                                  | 示例                                                   | 返回值                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------------------------------- | -------------------------------------- |
| [json_object](./json-constructor-functions/json_object.md) | 将一个或多个键值对转换为由这些键值对组成的 JSON 对象，键值对按照字典顺序排序。 | `SELECT JSON_OBJECT('Daniel Smith', 26, 'Lily Smith', 25);` | `{"Daniel Smith": 26, "Lily Smith": 25}` |
| [json_array](./json-constructor-functions/json_array.md) | 将 SQL 数组的每个元素转换为 JSON 值，并返回由这些 JSON 值组成的 JSON 数组。 | `SELECT JSON_ARRAY(1, 2, 3);`                                | `[1,2,3]`                                |
| [parse_json](./json-constructor-functions/parse_json.md) | 将字符串转换为 JSON 值。                           | `SELECT PARSE_JSON('{"a": 1}');`                             | `{"a": 1}`                               |

## JSON 查询函数和处理函数

JSON 查询函数和处理函数用于查询和处理 JSON 数据。例如，您可以使用路径表达式来定位 JSON 对象中的元素。

| 函数                                                     | 描述                                                  | 示例                                                    | 返回值                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- | ---------------------------------------------------------- |
| [arrow function](./json-query-and-processing-functions/arrow-function.md) | 查询 JSON 对象中可以通过路径表达式定位的元素。 | `SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';`                          | `1`                                                          |
| [cast](./json-query-and-processing-functions/cast.md) | 在 JSON 数据类型和 SQL 数据类型之间转换数据。 | `SELECT CAST(1 AS JSON);`                       | `1`      |
| [get_json_double](./json-query-and-processing-functions/get_json_double.md)   | 从 JSON 字符串的指定路径分析并获取浮点值。  | `SELECT get_json_double('{"k1":1.3, "k2":"2"}', "$.k1");` | `1.3` |
| [get_json_int](./json-query-and-processing-functions/get_json_int.md)   | 从 JSON 字符串的指定路径分析并获取整数值。  | `SELECT get_json_int('{"k1":1, "k2":"2"}', "$.k1");` | `1` |
| [get_json_string](./json-query-and-processing-functions/get_json_string.md)   | 从 JSON 字符串的指定路径分析并获取字符串。  | `SELECT get_json_string('{"k1":"v1", "k2":"v2"}', "$.k1");` | `v1` |
| [json_query](./json-query-and-processing-functions/json_query.md) | 查询 JSON 对象中可以通过路径表达式定位的元素的值。 | `SELECT JSON_QUERY('{"a": 1}', '$.a');`                         | `1`                                                   |
| [json_each](./json-query-and-processing-functions/json_each.md) | 将 JSON 对象的顶级元素扩展为键值对。 | `SELECT * FROM tj_test, LATERAL JSON_EACH(j);` | `!`[json_each](../../../assets/json_each.png) |
| [json_exists](./json-query-and-processing-functions/json_exists.md) | 检查 JSON 对象是否包含可以通过路径表达式定位的元素。如果该元素存在，则此函数返回 1。如果该元素不存在，则该函数返回 0。 | `SELECT JSON_EXISTS('{"a": 1}', '$.a'); `                      | `1`                                     |
| [json_keys](./json-query-and-processing-functions/json_keys.md) | 以 JSON 数组的形式返回 JSON 对象中的顶级键，或者，如果指定了路径，则返回路径中的顶级键。   | `SELECT JSON_KEYS('{"a": 1, "b": 2, "c": 3}');` |  `["a", "b", "c"]`|
| [json_length](./json-query-and-processing-functions/json_length.md) | 返回 JSON 文档的长度。  | `SELECT json_length('{"Name": "Alice"}');` |  `1`  |
| [json_string](./json-query-and-processing-functions/json_string.md)   | 将 JSON 对象转换为 JSON 字符串      | `SELECT json_string(parse_json('{"Name": "Alice"}'));` | `{"Name": "Alice"}`  |

## JSON 运算符

StarRocks 支持以下 JSON 比较运算符：`<`、`<=`、`>`、`>=`、`=` 和 `!=`。您可以使用这些运算符查询 JSON 数据。但是，不允许使用 `IN` 运算符查询 JSON 数据。有关 JSON 运算符的更多信息，请参见 [JSON 运算符](./json-operators.md)。

## JSON 路径表达式

您可以使用 JSON 路径表达式查询 JSON 对象中的元素。JSON 路径表达式的数据类型为 STRING。在大多数情况下，它们与各种 JSON 函数一起使用，例如 JSON_QUERY。在 StarRocks 中，JSON 路径表达式并不完全符合 [SQL/JSON 路径规范](https://modern-sql.com/blog/2017-06/whats-new-in-sql-2016#json-path)。有关 StarRocks 支持的 JSON 路径语法的信息，请参见下表，其中使用以下 JSON 对象作为示例。

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

| JSON 路径符号 | 描述                                                  | JSON 路径示例     | 返回值                                                 |
| ---------------- | ------------------------------------------------------------ | --------------------- | ------------------------------------------------------------ |
| `$`                | 表示根 JSON 对象。                                  | `'$'`                   | `{ "people": [ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ] }`|
|`.`               | 表示子 JSON 对象。                                 |`' $.people'`          |`[ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ]`|
|`[]`              | 表示一个或多个数组索引。 `[n]` 表示数组中的第 n 个元素。索引从 0 开始。 <br />StarRocks 2.5 支持查询多维数组，例如 . `["Lucy", "Daniel"], ["James", "Smith"]`要查询“Lucy”元素，可以使用 `$.people[0][0]`.| `'$.people [0]'`        | `{ "name": "Daniel", "surname": "Smith" }`                     |
| `[*]`             | 表示数组中的所有元素。                            | `'$.people[*].name'`    | `["Daniel", "Lily"]`                                           |
| `[start: end]`     | 表示数组中元素的子集。子集由区间指定`[start, end]`，区间不包括由结束索引表示的元素。 | `'$.people[0: 1].name'` | `["Daniel"]`                                                   |
