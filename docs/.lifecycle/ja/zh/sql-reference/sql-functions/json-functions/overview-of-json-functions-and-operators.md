---
displayed_sidebar: Chinese
---

# JSON 関数概要

StarRocks は以下の JSON コンストラクタ関数、JSON クエリおよび処理関数、JSON 演算子、および JSON オブジェクトの JSON Path をクエリする機能をサポートしています。

## JSON コンストラクタ関数

JSON コンストラクタ関数は、JSON タイプのデータを構築することができます。例えば、JSON タイプのオブジェクトや JSON タイプの配列などです。

| 関数名                                                       | 機能                                 | 例                                                        | 戻り値                                 |
| ------------------------------------------------------------ | ------------------------------------ | --------------------------------------------------------- | -------------------------------------- |
| [json_object](../../sql-functions/json-functions/json-constructor-functions/json_object.md) | JSON タイプのオブジェクトを構築します。 | `SELECT JSON_OBJECT('Daniel Smith', 26, 'Lily Smith', 25);` | `{"Daniel Smith": 26, "Lily Smith": 25}` |
| [json_array](../../sql-functions/json-functions/json-constructor-functions/json_array.md)   | JSON タイプの配列を構築します。       | `SELECT JSON_ARRAY(1, 2, 3);`                              | `[1,2,3]`                              |
| [parse_json](../../sql-functions/json-functions/json-constructor-functions/parse_json.md)   | 文字列から JSON タイプのデータを解析し構築します。 | `SELECT PARSE_JSON('{"a": 1}');`                           | `{"a": 1}`                             |

## JSON クエリおよび処理関数

JSON クエリおよび処理関数は、JSON タイプのデータをクエリおよび処理することができます。例えば、JSON オブジェクト内の特定のパスの値をクエリします。

| 関数名                                                       | 機能                                 | 例                                                        | 戻り値                                |
| ------------------------------------------------------------ | ------------------------------------ | --------------------------------------------------------- | -------------------------------------- |
| [arrow_function](../../sql-functions/json-functions/json-query-and-processing-functions/arrow-function.md) | JSON オブジェクト内の特定のパスの値をクエリします。 | `SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';`        | `1`                                   |
| [CAST AS JSON](../../sql-functions/json-functions/json-query-and-processing-functions/cast.md)| JSON タイプのデータと SQL タイプのデータを相互に変換します。 | `SELECT CAST(1 AS JSON);`                                 | `1`                                   |
| [get_json_double](../../sql-functions/json-functions/json-query-and-processing-functions/get_json_double.md)| json_str 内の json_path の浮動小数点数を解析し取得します。 | `SELECT get_json_double('{"k1":1.3, "k2":"2"}', "$.k1");` | `1.3`                                 |
| [get_json_int](../../sql-functions/json-functions/json-query-and-processing-functions/get_json_int.md)| json_str 内の json_path の整数を解析し取得します。 | `SELECT get_json_int('{"k1":1, "k2":"2"}', "$.k1");`      | `1`                                   |
| [get_json_string](../../sql-functions/json-functions/json-query-and-processing-functions/get_json_string.md)| json_str 内の json_path で指定された文字列を解析し取得します。この関数の別名は get_json_object です。 | `SELECT get_json_string('{"k1":"v1", "k2":"v2"}', "$.k1");` | `v1`                                 |
| [json_each](../../sql-functions/json-functions/json-query-and-processing-functions/json_each.md)   | 最外層の JSON オブジェクトをキーと値のペアに展開します。 | `SELECT * FROM tj_test, LATERAL JSON_EACH(j);`             | ![json_each](../../../assets/json_each.png) |
| [json_exists](../../sql-functions/json-functions/json-query-and-processing-functions/json_exists.md)| JSON オブジェクト内に特定の値が存在するかどうかをクエリします。存在する場合は 1 を、存在しない場合は 0 を返します。 | `SELECT JSON_EXISTS('{"a": 1}', '$.a');`                  | `1`                                   |
| [json_keys](../../sql-functions/json-functions/json-query-and-processing-functions/json_keys.md) | JSON オブジェクト内のすべての最上層のメンバー（キー）を配列として返します。 | `SELECT JSON_KEYS('{"a": 1, "b": 2, "c": 3}');`           | `["a", "b", "c"]`                     |
| [json_length](../../sql-functions/json-functions/json-query-and-processing-functions/json_length.md) | JSON 文字列の長さを返します。 | `SELECT json_length('{"Name": "Alice"}');`                | `1`                                   |
| [json_query](../../sql-functions/json-functions/json-query-and-processing-functions/json_query.md) | JSON オブジェクト内の特定のパスの値をクエリします。 | `SELECT JSON_QUERY('{"a": 1}', '$.a');`                   | `1`                                   |
| [json_string](../../sql-functions/json-functions/json-query-and-processing-functions/json_string.md)   | JSON オブジェクトを JSON 文字列に変換します。 | `SELECT json_string(parse_json('{"Name": "Alice"}'));`    | `{"Name": "Alice"}`                   |

## JSON 演算子

StarRocks は `<`、`<=`、`>`、`>=`、`=`、`!=` の演算子を使用して JSON データをクエリすることをサポートしており、`IN` 演算子はサポートしていません。JSON 演算子の詳細については、[JSON 演算子](../../sql-functions/json-functions/json-operators.md)を参照してください。

## JSON Path

JSON Path 式を使用して、JSON タイプのオブジェクト内の特定のパスの値をクエリすることができます。JSON Path は文字列タイプで、通常はさまざまな JSON 関数（例えば JSON_QUERY）と組み合わせて使用されます。現在、StarRocks での JSON Path は [SQL/JSONPath 標準](https://modern-sql.com/blog/2017-06/whats-new-in-sql-2016#json-path)に完全に準拠していません。StarRocks での JSON Path の文法については、以下の表を参照してください（以下の JSON オブジェクトを例にしています）。

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

| JSON Path の記号 | 説明                                                         | JSON Path の例       | 上記の JSON オブジェクトの値をクエリ                           |
| --------------- | ------------------------------------------------------------ | -------------------- | ------------------------------------------------------------ |
| $               | ルートノードのオブジェクトを表します。                        | '$'                  | `{ "people": [ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": "Smith", "active": true } ] }` |
| .               | 子ノードを表します。                                          | '$.people'           | `[ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": "Smith", "active": true } ]` |
| []              | 1つまたは複数の配列インデックスを表します。[n] は配列の n 番目の要素を選択します。0 から数え始めます。<br />**2.5 バージョンからは、多次元配列のクエリがサポートされています。例えば ["Lucy", "Daniel"], ["James", "Smith"]。"Lucy" をクエリするには、パス `$.people[0][0]` を使用します。**| '$.people[0]'        | `{ "name": "Daniel", "surname": "Smith"}` |
| [*]             | 配列内のすべての要素を表します。                             | '$.people[*].name'   | ["Daniel", "Lily"]                                            |
| [start: end]    | 配列の一部を表します。範囲は [start, end) で、end によって表される要素は含まれません。 | '$.people[0: 1].name' | ["Daniel"]                                                     |
