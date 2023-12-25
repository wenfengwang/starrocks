---
displayed_sidebar: English
---

# JSON 関数と演算子の概要

このトピックでは、StarRocks でサポートされている JSON コンストラクタ関数、クエリ関数、処理関数、演算子、パス式の概要について説明します。

## JSON コンストラクタ関数

JSON コンストラクタ関数は、JSON オブジェクトや JSON 配列などの JSON データを構築するために使用されます。

| 関数                                                     | 説明                                                  | 例                                                   | 戻り値                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------------------------------- | -------------------------------------- |
| [json_object](./json-constructor-functions/json_object.md) | 一つ以上のキーと値のペアを、キーと値のペアで構成される JSON オブジェクトに変換し、キーと値のペアを辞書順に並べ替えます。 | `SELECT JSON_OBJECT('Daniel Smith', 26, 'Lily Smith', 25);` | `{"Daniel Smith": 26, "Lily Smith": 25}` |
| [json_array](./json-constructor-functions/json_array.md) | SQL 配列の各要素を JSON 値に変換し、それらの JSON 値で構成される JSON 配列を返します。 | `SELECT JSON_ARRAY(1, 2, 3);`                                | `[1,2,3]`                                |
| [parse_json](./json-constructor-functions/parse_json.md) | 文字列を JSON 値に変換します。                           | `SELECT PARSE_JSON('{"a": 1}');`                             | `{"a": 1}`                               |

## JSON クエリ関数と処理関数

JSON クエリ関数と処理関数は、JSON データのクエリと処理に使用されます。たとえば、パス式を使用して、JSON オブジェクト内の要素を検索できます。

| 関数                                                     | 説明                                                  | 例                                                    | 戻り値                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- | ---------------------------------------------------------- |
| [arrow_function](./json-query-and-processing-functions/arrow-function.md) | JSON オブジェクト内のパス式で検索できる要素をクエリします。 | `SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';`                          | `1`                                                          |
| [cast](./json-query-and-processing-functions/cast.md) | JSON データ型と SQL データ型の間でデータを変換します。 | `SELECT CAST(1 AS JSON);`                       | `1`      |
| [get_json_double](./json-query-and-processing-functions/get_json_double.md)   | JSON 文字列内の指定されたパスから浮動小数点値を分析して取得します。  | `SELECT get_json_double('{"k1":1.3, "k2":"2"}', "$.k1");` | `1.3` |
| [get_json_int](./json-query-and-processing-functions/get_json_int.md)   | JSON 文字列内の指定されたパスから整数値を分析して取得します。  | `SELECT get_json_int('{"k1":1, "k2":"2"}', "$.k1");` | `1` |
| [get_json_string](./json-query-and-processing-functions/get_json_string.md)   | JSON 文字列内の指定されたパスから文字列を分析して取得します。  | `SELECT get_json_string('{"k1":"v1", "k2":"v2"}', "$.k1");` | `v1` |
| [json_query](./json-query-and-processing-functions/json_query.md) | JSON オブジェクト内のパス式で検索できる要素の値をクエリします。 | `SELECT JSON_QUERY('{"a": 1}', '$.a');`                         | `1`                                                   |
| [json_each](./json-query-and-processing-functions/json_each.md) | JSON オブジェクトの最上位の要素をキーと値のペアに展開します。 | `SELECT * FROM tj_test, LATERAL JSON_EACH(j);` | ![json_each](../../../assets/json_each.png) |
| [json_exists](./json-query-and-processing-functions/json_exists.md) | JSON オブジェクトに、パス式で検索できる要素が含まれているかどうかをチェックします。要素が存在する場合、この関数は 1 を返します。要素が存在しない場合、関数は 0 を返します。 | `SELECT JSON_EXISTS('{"a": 1}', '$.a'); `                      | `1`                                     |
| [json_keys](./json-query-and-processing-functions/json_keys.md) | JSON オブジェクトの最上位キーを JSON 配列として返すか、パスが指定されている場合はパスの最上位キーを返します。   | `SELECT JSON_KEYS('{"a": 1, "b": 2, "c": 3}');` |  `["a", "b", "c"]`|
| [json_length](./json-query-and-processing-functions/json_length.md) | JSON ドキュメントの長さを返します。  | `SELECT json_length('{"Name": "Alice"}');` |  `1`  |
| [json_string](./json-query-and-processing-functions/json_string.md)   | JSON オブジェクトを JSON 文字列に変換します      | `SELECT json_string(parse_json('{"Name": "Alice"}'));` | `{"Name": "Alice"}`  |

## JSON 演算子

StarRocks は、`<`、 `<=`, `>`、 `>=`、 `=`、 `!=` の JSON 比較演算子をサポートしています。これらの演算子を使用して、JSON データをクエリできます。ただし、`IN` を使用して JSON データのクエリを実行することはできません。JSON 演算子の詳細については、[JSON 演算子](./json-operators.md)を参照してください。

## JSON パス式

JSON パス式を使用して、JSON オブジェクト内の要素をクエリできます。JSON パス式は STRING データ型です。ほとんどの場合、JSON_QUERY などのさまざまな JSON 関数で使用されます。StarRocksでは、JSON パス式は [SQL/JSON パス仕様](https://modern-sql.com/blog/2017-06/whats-new-in-sql-2016#json-path)に完全に準拠しているわけではありません。StarRocks でサポートされている JSON パス構文については、次の JSON オブジェクトを例として使用している次の表を参照してください。

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

| JSON パス記号 | 説明                                                  | JSON パスの例     | 戻り値                                                 |
| ---------------- | ------------------------------------------------------------ | --------------------- | ------------------------------------------------------------ |
| `$`                | ルート JSON オブジェクトを示します。                                  | `'$'`                   | `{ "people": [ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": "Smith", "active": true } ] }`|
| `.`               | 子 JSON オブジェクトを示します。                                 | `'$.people'`          | `[ { "name": "Daniel", "surname": "Smith" }, { "name": "Lily", "surname": "Smith", "active": true } ]`|
| `[]`              | 一つ以上の配列インデックスを示します。 `[n]` は配列の n 番目の要素を示します。インデックスは 0 から始まります。 <br />StarRocks 2.5 は、多次元配列のクエリをサポートしています。例えば `["Lucy", "Daniel"], ["James", "Smith"]` で "Lucy" 要素をクエリするには、`$.people[0][0]` を使用します。| `'$.people[0]'`        | `{ "name": "Daniel", "surname": "Smith" }`                     |
| `[*]`             | 配列内のすべての要素を示します。                            | `'$.people[*].name'`    | `["Daniel", "Lily"]`                                           |
| `[start:end]`     | 配列の要素のサブセットを示します。サブセットは、`[start:end]` の間隔によって指定され、終了インデックスで示される要素を除外します。| `'$.people[0:1].name'` | `["Daniel"]`                                                   |
