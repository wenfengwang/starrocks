---
displayed_sidebar: "Japanese"
---

# json_exists

## 説明

JSONオブジェクトに`json_path`式で指定された要素が含まれているかどうかをチェックします。要素が存在する場合、JSON_EXISTS関数は`1`を返します。それ以外の場合、JSON_EXISTS関数は`0`を返します。

## 構文

```Haskell
json_exists(json_object_expr, json_path)
```

## パラメータ

- `json_object_expr`: JSONオブジェクトを表す式。オブジェクトはJSON列、またはPARSE_JSONなどのJSONコンストラクタ関数によって生成されるJSONオブジェクトであることができます。

- `json_path`: JSONオブジェクト内の要素へのパスを表す式。このパラメータの値は文字列です。StarRocksでサポートされているJSONパス構文の詳細については、「[JSON関数および演算子の概要](../overview-of-json-functions-and-operators.md)」を参照してください。

## 戻り値

BOOLEAN値を返します。

## 例

例1: 指定されたJSONオブジェクトに`'$.a.b'`式で指定された要素が含まれているかどうかをチェックします。この例では、JSONオブジェクトに要素が存在します。したがって、json_exists関数は`1`を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

例2: 指定されたJSONオブジェクトに`'$.a.c'`式で指定された要素が含まれているかどうかをチェックします。この例では、JSONオブジェクトに要素が存在しません。したがって、json_exists関数は`0`を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> 0
```

例3: 指定されたJSONオブジェクトに`'$.a[2]'`式で指定された要素が含まれているかどうかをチェックします。この例では、配列aという名前のJSONオブジェクトにはインデックス2の要素が含まれています。したがって、json_exists関数は`1`を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 1
```

例4: 指定されたJSONオブジェクトに`'$.a[3]'`式で指定された要素が含まれているかどうかをチェックします。この例では、配列aという名前のJSONオブジェクトにはインデックス3の要素が含まれていません。したがって、json_exists関数は`0`を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> 0
```