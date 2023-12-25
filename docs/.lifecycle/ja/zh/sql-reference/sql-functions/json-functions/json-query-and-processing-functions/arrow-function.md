---
displayed_sidebar: Chinese
---

# アロー関数

## 機能

アロー関数 `->` は、JSON オブジェクト内の指定されたパス(`json_path`)の値を検索し、JSON 型として出力します。アロー関数は JSON_QUERY 関数よりも簡潔で使いやすいです。

## 文法

```Plain Text
json_object_expr -> json_path
```

## パラメータ説明

- `json_object_expr`: JSON オブジェクトの式で、JSON 型の列や PARSE_JSON などの JSON 関数で構築された JSON オブジェクトが使用できます。

- `json_path`: JSON オブジェクトを検索する際のパス。サポートされるデータ型は文字列です。StarRocks がサポートする JSON Path の文法については、[JSON Path 文法](../overview-of-json-functions-and-operators.md#json-path)を参照してください。

## 戻り値の説明

JSON 型の値を返します。

> 検索したフィールドが存在しない場合は、SQL 型の NULL を返します。

## 例

例 1: JSON オブジェクト内のパス式 `'$.a.b'` で指定された値を検索します。

```Plain Text
mysql> SELECT PARSE_JSON('{"a": {"b": 1}}') -> '$.a.b';
       -> 1
```

例 2: アロー関数をネストして使用し、前のアロー関数の結果に基づいて検索を行います。

> この例では `json_path` からルート要素$を省略しています。

```Plain Text
mysql> SELECT PARSE_JSON('{"a": {"b": 1}}')->'a'->'b';
       -> 1
```

例 3: JSON オブジェクト内のパス式 `'a'` で指定された値を検索します。

> この例では `json_path` からルート要素$を省略しています。

```Plain Text
mysql> SELECT PARSE_JSON('{"a": "b"}') -> 'a';
       -> "b"
```
