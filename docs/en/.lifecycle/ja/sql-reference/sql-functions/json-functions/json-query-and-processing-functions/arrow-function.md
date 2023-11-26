---
displayed_sidebar: "Japanese"
---

# Arrow function（アロー関数）

## 説明

JSONオブジェクト内の`json_path`式で指定された要素をクエリし、JSON値を返します。アロー関数`->`は、[json_query](json_query.md)関数よりもコンパクトで使いやすいです。

## 構文

```Haskell
json_object_expr -> json_path
```

## パラメーター

- `json_object_expr`：JSONオブジェクトを表す式です。オブジェクトはJSON列またはPARSE_JSONなどのJSONコンストラクタ関数によって生成されるJSONオブジェクトです。

- `json_path`：JSONオブジェクト内の要素へのパスを表す式です。このパラメーターの値は文字列です。StarRocksでサポートされているJSONパスの構文については、[JSON関数および演算子の概要](../overview-of-json-functions-and-operators.md)を参照してください。

## 戻り値

JSON値を返します。

> 要素が存在しない場合、アロー関数は`NULL`のSQL値を返します。

## 例

例1：指定されたJSONオブジェクト内の`'$.a.b'`式で指定された要素をクエリします。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';

       -> 1
```

例2：ネストされたアロー関数を使用して要素をクエリします。別のアロー関数がネストされたアロー関数内で、ネストされたアロー関数によって返された結果に基づいて要素をクエリします。

> この例では、`json_path`式からルート要素$が省略されています。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}')->'a'->'b';

       -> 1
```

例3：指定されたJSONオブジェクト内の`'a'`式で指定された要素をクエリします。

> この例では、`json_path`式からルート要素$が省略されています。

```plaintext
mysql> SELECT parse_json('{"a": "b"}') -> 'a';

       -> "b"
```
