```yaml
---
displayed_sidebar: "Japanese"
---

# JSON 関数および演算子の概要

このトピックでは、StarRocks でサポートされている JSON コンストラクタ関数、クエリ関数、処理関数、演算子、およびパス表現の概要を提供します。

## JSON コンストラクタ関数

JSON コンストラクタ関数は、JSON オブジェクトや JSON 配列などの JSON データを構築するために使用されます。

| 関数                                                     | 説明                                                  | 例                                              | 戻り値                             |
| -------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------ | ---------------------------------- |
| [json_object](./json-constructor-functions/json_object.md) | 1 つ以上のキーと値のペアを、キーで辞書順に並べ替えられた JSON オブジェクトに変換します。 | `SELECT JSON_OBJECT(' Daniel Smith', 26, 'Lily Smith', 25);` | `{"Daniel Smith": 26, "Lily Smith": 25}` |
| [json_array](./json-constructor-functions/json_array.md) | SQL 配列の各要素を JSON 値に変換し、それらの JSON 値で構成される JSON 配列を返します。 | `SELECT JSON_ARRAY(1, 2, 3);`                                | `[1,2,3]`                                |
| [parse_json](./json-constructor-functions/parse_json.md) | 文字列を JSON 値に変換します。                           | `SELECT PARSE_JSON('{"a": 1}');`                             | `{"a": 1}`                               |

## JSON クエリ関数および処理関数

JSON クエリ関数および処理関数は、JSON データのクエリと処理に使用されます。たとえば、JSON オブジェクト内の要素を特定するためにパス表現を使用できます。

| 関数                                                     | 説明                                                  | 例                                                    | 戻り値                                               |
| -------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------ |
| [arrow function](./json-query-and-processing-functions/arrow-function.md) | JSON オブジェクト内のパス表現で特定できる要素をクエリします。 | `SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';`                          | `1`                                                          |
| [cast](./json-query-and-processing-functions/cast.md) | JSON データ型と SQL データ型の間でデータを変換します。 | `SELECT CAST(1 AS JSON);`                       | `1`      |
| [get_json_double](./json-query-and-processing-functions/get_json_double.md)   | JSON 文字列内の指定されたパスから浮動小数点値を解析および取得します。  | `SELECT GET_JSON_DOUBLE('{"k1":1.3, "k2":"2"}', "$.k1");` | `1.3` |
| [get_json_int](./json-query-and-processing-functions/get_json_int.md)   | JSON 文字列内の指定されたパスから整数値を解析および取得します。  | `SELECT GET_JSON_INT('{"k1":1, "k2":"2"}', "$.k1");` | `1` |
| [get_json_string](./json-query-and-processing-functions/get_json_string.md)   | JSON 文字列内の指定されたパスから文字列を解析および取得します。  | `SELECT GET_JSON_STRING('{"k1":"v1", "k2":"v2"}', "$.k1");` | `v1` |
| [json_query](./json-query-and-processing-functions/json_query.md) | JSON オブジェクト内のパス表現で特定できる要素の値をクエリします。 | `SELECT JSON_QUERY('{"a": 1}', '$.a');`                         | `1`                                                   |
| [json_each](./json-query-and-processing-functions/json_each.md) | JSON オブジェクトのトップレベル要素をキーと値のペアに展開します。 | `SELECT * FROM tj_test, LATERAL JSON_EACH(j);` | `!`[json_each](../../../assets/json_each.png) |
| [json_exists](./json-query-and-processing-functions/json_exists.md) | JSON オブジェクトに、パス表現で特定できる要素が含まれているかどうかを確認します。要素が存在する場合、この関数は 1 を返します。要素が存在しない場合、関数は 0 を返します。 | `SELECT JSON_EXISTS('{"a": 1}', '$.a'); `                      | `1`                                     |
| [json_keys](./json-query-and-processing-functions/json_keys.md) | JSON オブジェクトのトップレベルのキーを JSON 配列として返します。または、パスが指定されている場合は、そのパスのトップレベルのキーを返します。   | `SELECT JSON_KEYS('{"a": 1, "b": 2, "c": 3}');` |  `["a", "b", "c"]`|
| [json_length](./json-query-and-processing-functions/json_length.md) | JSON ドキュメントの長さを返します。  | `SELECT JSON_LENGTH('{"Name": "Alice"}');` |  `1`  |
| [json_string](./json-query-and-processing-functions/json_string.md)   | JSON オブジェクトを JSON 文字列に変換します  | `SELECT JSON_STRING(PARSE_JSON('{"Name": "Alice"}'));` | `{"Name": "Alice"}`  |

## JSON 演算子

StarRocks は、以下の JSON 比較演算子をサポートしています: `<`, `<=`, `>`, `>=`, `=`, および `!=`。これらの演算子を使用して JSON データをクエリできます。ただし、`IN` を使用して JSON データをクエリすることはできません。JSON 演算子の詳細については、[JSON 演算子](./json-operators.md)を参照してください。

## JSON パス表現

JSON パス表現を使用して、JSON オブジェクト内の要素をクエリできます。JSON パス表現は STRING データ型です。ほとんどの場合、これらは JSON_QUERY などの様々な JSON 関数とともに使用されます。StarRocks では、JSON パス表現は [SQL/JSON パスの仕様](https://modern-sql.com/blog/2017-06/whats-new-in-sql-2016#json-path)と完全に準拠していません。StarRocks でサポートされている JSON パス構文については、次の表を参照してください。この表では、次の JSON オブジェクトが例として使用されています。

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

| JSON パス記号 | 説明                                                  | JSON パス例     | 戻り値                                               |
| ------------- | ------------------------------------------------- | --------------- | ------------------------------------------------------ |
| `$`            | ルート JSON オブジェクトを示します。               | `'$'`           | `{ "people": [ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ] }`|
|`.`             | 子 JSON オブジェクトを示します。                   |`'$.people'`     |`[ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ]`|
|`[]`           | 1 つ以上の配列インデックスを示します。`[n]` は配列内の n 番目の要素を示します。インデックスは 0 から始まります。<br />StarRocks 2.5 では、多次元配列をクエリすることができます。たとえば、`["Lucy", "Daniel"], ["James", "Smith"]` をクエリするには `$.people[0][0]` を使用できます。| `'$.people[0]'` | `{ "name": "Daniel", "surname": "Smith" }`         |
| `[*]`         | 配列内のすべての要素を示します。                | `'$.people[*].name'` | `["Daniel", "Lily"]`                              |
| `[start: end]`| 配列内の要素の部分集合を示します。部分集合は `[start, end]` 間隔で指定され、end インデックスで示される要素は除外されます。 | `'$.people[0: 1].name'` | `["Daniel"]`                                           |
```