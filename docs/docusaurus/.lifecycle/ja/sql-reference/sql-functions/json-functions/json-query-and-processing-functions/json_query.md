---
displayed_sidebar: "Japanese"
---

# json_query

## 説明

JSONオブジェクト内で`json_path`式によって位置付けられる要素の値をクエリし、JSON値を返します。

## 構文

```Haskell
json_query(json_object_expr, json_path)
```

## パラメータ

- `json_object_expr`: JSONオブジェクトを表す式です。オブジェクトはJSON列であってもよし、PARSE_JSONなどのJSONコンストラクタ関数によって生成されたJSONオブジェクトであってもよい。

- `json_path`: JSONオブジェクト内の要素へのパスを表す式です。このパラメータの値は文字列です。StarRocksがサポートするJSONパス構文の詳細については、[JSON関数および演算子の概要](../overview-of-json-functions-and-operators.md)を参照してください。

## 戻り値

JSON値を返します。

> 要素が存在しない場合、json_query関数は`NULL`のSQL値を返します。

## 例

例1: 指定されたJSONオブジェクト内で`'$.a.b'`式によって位置付けられる要素の値をクエリします。この例では、json_query関数はJSON値の`1`を返します。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

例2: 指定されたJSONオブジェクト内で`'$.a.c'`式によって位置付けられる要素の値をクエリします。この例では要素が存在しないため、json_query関数は`NULL`のSQL値を返します。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> NULL
```

例3: 指定されたJSONオブジェクト内で`'$.a[2]'`式によって位置付けられる要素の値をクエリします。この例では、配列であるaという名前のJSONオブジェクトに、インデックス2の要素が含まれており、その値が3であるため、JSON_QUERY関数はJSON値の`3`を返します。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 3
```

例4: 指定されたJSONオブジェクト内で`'$.a[3]'`式によって位置付けられる要素をクエリします。この例では、配列であるaという名前のJSONオブジェクトに、インデックス3の要素が含まれていないため、json_query関数は`NULL`のSQL値を返します。

```plaintext
mysql> SELECT json_query(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> NULL
```