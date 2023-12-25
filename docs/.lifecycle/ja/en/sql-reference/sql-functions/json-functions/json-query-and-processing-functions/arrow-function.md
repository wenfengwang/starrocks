---
displayed_sidebar: English
---

# アロー関数

## 説明

JSONオブジェクト内で`json_path`式によって位置を特定できる要素をクエリし、JSON値を返します。アロー関数`->`は、[json_query](json_query.md)関数よりもコンパクトで使いやすいです。

## 構文

```Haskell
json_object_expr -> json_path
```

## パラメーター

- `json_object_expr`: JSONオブジェクトを表す式。このオブジェクトはJSONカラムであるか、PARSE_JSONのようなJSONコンストラクタ関数によって生成されたJSONオブジェクトです。

- `json_path`: JSONオブジェクト内の要素へのパスを表す式。このパラメータの値は文字列です。StarRocksがサポートするJSONパス構文については、[JSON関数と演算子の概要](../overview-of-json-functions-and-operators.md)を参照してください。

## 戻り値

JSON値を返します。

> 要素が存在しない場合、アロー関数はSQL値の`NULL`を返します。

## 例

例1: 指定されたJSONオブジェクト内で`'$.a.b'`式によって位置を特定できる要素をクエリします。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';

       -> 1
```

例2: 入れ子になったアロー関数を使用して要素をクエリします。他のアロー関数が入れ子になっているアロー関数は、入れ子になったアロー関数によって返される結果に基づいて要素をクエリします。

> この例では、ルート要素$は`json_path`式から省略されています。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}')->'a'->'b';

       -> 1
```

例3: 指定されたJSONオブジェクト内で`'a'`式によって位置を特定できる要素をクエリします。

> この例では、ルート要素$は`json_path`式から省略されています。

```plaintext
mysql> SELECT parse_json('{"a": "b"}') -> 'a';

       -> "b"
```
