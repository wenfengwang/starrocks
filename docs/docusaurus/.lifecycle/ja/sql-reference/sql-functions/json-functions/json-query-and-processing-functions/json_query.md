---
displayed_sidebar: "Japanese"
---

# json_query

## 説明

JSONオブジェクト内の`json_path`式で特定される要素の値をクエリし、JSON値を返します。

## 構文

```Haskell
json_query(json_object_expr, json_path)
```

## パラメーター

- `json_object_expr`: JSONオブジェクトを表す式です。オブジェクトはJSON列、またはPARSE_JSONなどのJSONコンストラクタ関数によって生成されたJSONオブジェクトです。

- `json_path`: JSONオブジェクト内の要素へのパスを表す式です。このパラメーターの値は文字列です。StarRocksでサポートされているJSONパス構文の詳細については、[JSON関数および演算子の概要](../overview-of-json-functions-and-operators.md)を参照してください。

## 戻り値

JSON値を返します。

> 要素が存在しない場合、json_query関数は`NULL`のSQL値を返します。

## 例

例1: 指定されたJSONオブジェクト内で`'$.a.b'`式で特定される要素の値をクエリします。この例では、json_query関数はJSON値`1`を返します。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

例2: 指定されたJSONオブジェクト内で`'$.a.c'`式で特定される要素の値をクエリします。この例では、要素は存在しないため、json_query関数はSQL値の`NULL`を返します。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> NULL
```

例3: 指定されたJSONオブジェクト内で`'$.a[2]'`式で特定される要素の値をクエリします。この例では、要素が3である要素を含む配列aという名前のJSONオブジェクトがあります。したがって、JSON_QUERY関数はJSON値`3`を返します。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 3
```

例4: 指定されたJSONオブジェクト内で`'$.a[3]'`式で特定される要素をクエリします。この例では、配列aという名前のJSONオブジェクトにはインデックス3の要素が含まれていません。したがって、json_query関数はSQL値の`NULL`を返します。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> NULL
```