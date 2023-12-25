---
displayed_sidebar: English
---

# json_exists

## 説明

JSON オブジェクトに `json_path` 式で位置を特定できる要素が含まれているかどうかをチェックします。要素が存在する場合、`json_exists` 関数は `1` を返します。存在しない場合は `0` を返します。

## 構文

```Haskell
json_exists(json_object_expr, json_path)
```

## パラメーター

- `json_object_expr`: JSON オブジェクトを表す式です。オブジェクトは、JSON 列や PARSE_JSON などの JSON コンストラクタ関数によって生成された JSON オブジェクトである可能性があります。

- `json_path`: JSON オブジェクト内の要素へのパスを表す式です。このパラメータの値は文字列です。StarRocks がサポートする JSON パス構文の詳細については、[JSON 関数と演算子の概要](../overview-of-json-functions-and-operators.md)を参照してください。

## 戻り値

BOOLEAN 値を返します。

## 例

例 1: 指定された JSON オブジェクトに `'$.a.b'` 式で位置を特定できる要素が含まれているかどうかを確認します。この例では、要素は JSON オブジェクトに存在します。したがって、`json_exists` 関数は `1` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

例 2: 指定された JSON オブジェクトに `'$.a.c'` 式で位置を特定できる要素が含まれているかどうかを確認します。この例では、要素は JSON オブジェクトに存在しません。したがって、`json_exists` 関数は `0` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> 0
```

例 3: 指定された JSON オブジェクトに `'$.a[2]'` 式で位置を特定できる要素が含まれているかどうかを確認します。この例では、JSON オブジェクト（a という名前の配列）にはインデックス 2 の要素が含まれています。したがって、`json_exists` 関数は `1` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 1
```

例 4: 指定された JSON オブジェクトに `'$.a[3]'` 式で位置を特定できる要素が含まれているかどうかを確認します。この例では、JSON オブジェクト（a という名前の配列）にはインデックス 3 の要素が含まれていません。したがって、`json_exists` 関数は `0` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> 0
```
