---
displayed_sidebar: "Japanese"
---

# Arrow function

## 説明

JSONオブジェクト内の`json_path`式で指定された要素をクエリし、JSON値を返します。 矢印関数 `->` は、[json_query](json_query.md) 関数よりもコンパクトで使いやすいです。

## 構文

```Haskell
json_object_expr -> json_path
```

## パラメーター

- `json_object_expr`: JSONオブジェクトを表す式。 このオブジェクトは、JSON列、またはPARSE_JSONなどのJSONコンストラクタ関数によって生成されるJSONオブジェクトであることができます。

- `json_path`: JSONオブジェクト内の要素へのパスを表す式。 このパラメータの値は文字列です。 StarRocksがサポートするJSONパス構文の詳細については、[JSON関数と演算子の概要](../overview-of-json-functions-and-operators.md) を参照してください。

## 戻り値

JSON値を返します。

> 要素が存在しない場合、矢印関数は`NULL`のSQL値を返します。

## 例

例 1: 指定されたJSONオブジェクト内の `'$.a.b'` 式で見つけることができる要素を問い合わせます。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';

       -> 1
```

例 2: ネストされた矢印関数を使用して要素をクエリします。 ネストされた矢印関数が含まれる矢印関数は、ネストされた矢印関数によって返される結果に基づいて要素をクエリします。

> この例では、ルート要素 $ が `json_path` 式から省略されています。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}')->'a'->'b';

       -> 1
```

例 3: 指定されたJSONオブジェクト内の `'a'` 式で見つけることができる要素を問い合わせます。

> この例では、ルート要素 $ が `json_path` 式から省略されています。

```plaintext
mysql> SELECT parse_json('{"a": "b"}') -> 'a';

       -> "b"
```