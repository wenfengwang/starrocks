---
displayed_sidebar: "Japanese"
---

# JSON関数と演算子の概要

このトピックでは、StarRocksでサポートされているJSONコンストラクタ関数、クエリ関数、処理関数、演算子、およびパス式の概要を提供します。

## JSONコンストラクタ関数

JSONコンストラクタ関数は、JSONオブジェクトやJSON配列などのJSONデータを構築するために使用されます。

| 関数                                                          | 説明                                                         | 例                                                    | 戻り値                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ | -------------------------------------- |
| [json_object](./json-constructor-functions/json_object.md) | 1つ以上のキーと値のペアを、キーで辞書順にソートされたキーと値のペアで構成されるJSONオブジェクトに変換します。 | `SELECT JSON_OBJECT(' Daniel Smith', 26, 'Lily Smith', 25);` | `{"Daniel Smith": 26, "Lily Smith": 25}` |
| [json_array](./json-constructor-functions/json_array.md) | SQLの配列の各要素をJSON値に変換し、それらのJSON値で構成されるJSON配列を返します。 | `SELECT JSON_ARRAY(1, 2, 3);`                                | `[1,2,3]`                                |
| [parse_json](./json-constructor-functions/parse_json.md) | 文字列をJSON値に変換します。                           | `SELECT PARSE_JSON('{"a": 1}');`                             | `{"a": 1}`                               |

## JSONクエリ関数と処理関数

JSONクエリ関数と処理関数は、JSONデータのクエリと処理に使用されます。たとえば、JSONオブジェクト内の要素を指定するためにパス式を使用できます。

| 関数                                                     | 説明                                                         | 例                                                       | 戻り値                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- | ---------------------------------------------------------- |
| [arrow function](./json-query-and-processing-functions/arrow-function.md) | JSONオブジェクト内のパス式で指定できる要素をクエリします。 | `SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';`                          | `1`                                                          |
| [cast](./json-query-and-processing-functions/cast.md) | JSONデータ型とSQLデータ型の間でデータを変換します。 | `SELECT CAST(1 AS JSON);`                       | `1`      |
| [get_json_double](./json-query-and-processing-functions/get_json_double.md)   | JSON文字列内の指定されたパスから浮動小数点値を解析して取得します。  | `SELECT get_json_double('{"k1":1.3, "k2":"2"}', "$.k1");` | `1.3` |
| [get_json_int](./json-query-and-processing-functions/get_json_int.md)   | JSON文字列内の指定されたパスから整数値を解析して取得します。  | `SELECT get_json_int('{"k1":1, "k2":"2"}', "$.k1");` | `1` |
| [get_json_string](./json-query-and-processing-functions/get_json_string.md)   | JSON文字列内の指定されたパスから文字列を解析して取得します。  | `SELECT get_json_string('{"k1":"v1", "k2":"v2"}', "$.k1");` | `v1` |
| [json_query](./json-query-and-processing-functions/json_query.md) | JSONオブジェクト内のパス式で指定できる要素の値をクエリします。 | `SELECT JSON_QUERY('{"a": 1}', '$.a');`                         | `1`                                                   |
| [json_each](./json-query-and-processing-functions/json_each.md) | JSONオブジェクトのトップレベル要素をキーと値のペアに展開します。 | `SELECT * FROM tj_test, LATERAL JSON_EACH(j);` | `!`[json_each](../../../assets/json_each.png) |
| [json_exists](./json-query-and-processing-functions/json_exists.md) | JSONオブジェクトがパス式で指定できる要素を含むかどうかをチェックします。要素が存在する場合、この関数は1を返し、存在しない場合は0を返します。 | `SELECT JSON_EXISTS('{"a": 1}', '$.a'); `                      | `1`                                     |
| [json_keys](./json-query-and-processing-functions/json_keys.md) | JSONオブジェクトのトップレベルのキーをJSON配列として返します。または、パスが指定されている場合、そのパスのトップレベルのキーを返します。   | `SELECT JSON_KEYS('{"a": 1, "b": 2, "c": 3}');` |  `["a", "b", "c"]`|
| [json_length](./json-query-and-processing-functions/json_length.md) | JSONドキュメントの長さを返します。  | `SELECT json_length('{"Name": "Alice"}');` |  `1`  |
| [json_string](./json-query-and-processing-functions/json_string.md)   | JSONオブジェクトをJSON文字列に変換します      | `SELECT json_string(parse_json('{"Name": "Alice"}'));` | `{"Name": "Alice"}`  |

## JSON演算子

StarRocksは、次のJSON比較演算子をサポートしています：`<`, `<=`, `>`, `>=`, `=`, および `!=`。これらの演算子を使用してJSONデータをクエリできます。ただし、`IN`を使用してJSONデータをクエリすることはできません。JSON演算子の詳細については、[JSON演算子](./json-operators.md)を参照してください。

## JSONパス式

JSONパス式を使用して、JSONオブジェクト内の要素をクエリできます。JSONパス式はSTRINGデータ型です。ほとんどの場合、これらはJSON_QUERYなどのさまざまなJSON関数とともに使用されます。StarRocksでは、JSONパス式は完全に[SQL/JSONパスの仕様](https://modern-sql.com/blog/2017-06/whats-new-in-sql-2016#json-path)に準拠していません。StarRocksでサポートされているJSONパス構文に関する詳細については、以下の表を参照してください。この表では、次のJSONオブジェクトが例として使用されています。

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

| JSONパス記号 | 説明                                                         | JSONパスの例     | 戻り値                                                 |
| ---------------- | ------------------------------------------------------------ | --------------------- | ------------------------------------------------------------ |
| `$`                | ルートのJSONオブジェクトを示します。                                  | `'$'`                   | `{ "people": [ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ] }`|
|`.`               | 子のJSONオブジェクトを示します。                                 |`' $.people'`          |`[ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": Smith, "active": true } ]`|
|`[]`              | 1つ以上の配列インデックスを示します。 `[n]`は配列内のn番目の要素を示し、インデックスは0から始まります。<br />StarRocks 2.5では、多次元配列のクエリをサポートしており、たとえば、`["Lucy", "Daniel"], ["James", "Smith"]` のような形式です。"Lucy"要素をクエリするには、`$.people[0][0]` を使用できます。| `'$.people [0]'`        | `{ "name": "Daniel", "surname": "Smith" }`                     |
| `[*]`             | 配列内のすべての要素を示します。                            | `'$.people[*].name'`    | `["Daniel", "Lily"]`                                           |
| `[start: end]`     | 配列からの要素の部分集合を示します。部分集合は`[start, end]`間隔で指定され、終了インデックスで示されている要素は除外されます。 | `'$.people[0: 1].name'` | `["Daniel"]`                                                   |