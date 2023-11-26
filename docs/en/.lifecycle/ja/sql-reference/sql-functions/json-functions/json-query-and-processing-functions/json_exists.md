---
displayed_sidebar: "Japanese"
---

# json_exists

## 説明

`json_path` 式で指定された要素を含むかどうかをチェックします。要素が存在する場合、JSON_EXISTS 関数は `1` を返します。存在しない場合、JSON_EXISTS 関数は `0` を返します。

## 構文

```Haskell
json_exists(json_object_expr, json_path)
```

## パラメータ

- `json_object_expr`: JSON オブジェクトを表す式です。オブジェクトは JSON カラムまたは PARSE_JSON のような JSON コンストラクタ関数によって生成された JSON オブジェクトです。

- `json_path`: JSON オブジェクト内の要素へのパスを表す式です。このパラメータの値は文字列です。StarRocks でサポートされている JSON パスの構文の詳細については、[JSON 関数および演算子の概要](../overview-of-json-functions-and-operators.md)を参照してください。

## 戻り値

BOOLEAN 値を返します。

## 例

例 1: 指定された JSON オブジェクトが `'$.a.b'` 式で指定された要素を含むかどうかをチェックします。この例では、要素が JSON オブジェクトに存在します。したがって、json_exists 関数は `1` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;

       -> 1
```

例 2: 指定された JSON オブジェクトが `'$.a.c'` 式で指定された要素を含むかどうかをチェックします。この例では、要素が JSON オブジェクトに存在しません。したがって、json_exists 関数は `0` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;

       -> 0
```

例 3: 指定された JSON オブジェクトが `'$.a[2]'` 式で指定された要素を含むかどうかをチェックします。この例では、配列 a という名前の JSON オブジェクトにはインデックス 2 の要素が含まれています。したがって、json_exists 関数は `1` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;

       -> 1
```

例 4: 指定された JSON オブジェクトが `'$.a[3]'` 式で指定された要素を含むかどうかをチェックします。この例では、配列 a という名前の JSON オブジェクトにはインデックス 3 の要素が含まれていません。したがって、json_exists 関数は `0` を返します。

```plaintext
mysql> SELECT json_exists(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;

       -> 0
```
