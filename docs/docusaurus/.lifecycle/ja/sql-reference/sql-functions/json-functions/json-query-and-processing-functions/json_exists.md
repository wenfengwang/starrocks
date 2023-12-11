---
displayed_sidebar: "Japanese"
---

# json_exists

## 説明

JSONオブジェクトに、`json_path` 式で指定された要素が含まれているかどうかをチェックします。要素が存在する場合、JSON_EXISTS 関数は `1` を返します。存在しない場合は `0` を返します。

## 構文

```Haskell
json_exists(json_object_expr, json_path)
```

## パラメーター

- `json_object_expr`: JSONオブジェクトを表す式。このオブジェクトは、JSON列やPARSE_JSONなどのJSONコンストラクタ関数によって生成されたJSONオブジェクトを指すことができます。

- `json_path`: JSONオブジェクト内の要素へのパスを表す式。このパラメーターの値は文字列です。StarRocksでサポートされているJSONパス構文の詳細については、[JSON関数および演算子の概要](../overview-of-json-functions-and-operators.md)を参照してください。

## 戻り値

BOOLEAN値を返します。

## 例

例1: 指定されたJSONオブジェクトが`'$.a.b'` 式で指定された要素を含んでいるかどうかをチェックします。この例では、JSONオブジェクトに要素が存在しているため、json_exists 関数は `1` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

例2: 指定されたJSONオブジェクトが`'$.a.c'` 式で指定された要素を含んでいるかどうかをチェックします。この例では、JSONオブジェクトに要素が存在しないため、json_exists 関数は `0` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> 0
```

例3: 指定されたJSONオブジェクトが`'$.a[2]'` 式で指定された要素を含んでいるかどうかをチェックします。この例では、配列aという名前のJSONオブジェクトに、インデックス2の要素が含まれています。したがって、json_exists 関数は `1` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 1
```

例4: 指定されたJSONオブジェクトが`'$.a[3]'` 式で指定された要素を含んでいるかどうかをチェックします。この例では、配列aという名前のJSONオブジェクトに、インデックス3の要素が含まれていません。したがって、json_exists 関数は `0` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> 0
```