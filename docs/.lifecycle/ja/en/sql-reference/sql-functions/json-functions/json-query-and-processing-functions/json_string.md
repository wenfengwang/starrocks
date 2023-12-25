---
displayed_sidebar: English
---

# json_string

## 説明

JSONオブジェクトをJSON文字列に変換します。

## 構文

```SQL
json_string(json_object_expr)
```

## パラメーター

- `json_object_expr`: JSONオブジェクトを表す式です。オブジェクトは、JSON列やPARSE_JSONのようなJSONコンストラクタ関数によって生成されたJSONオブジェクトである可能性があります。

## 戻り値

VARCHAR型の値を返します。

## 例

例 1: JSONオブジェクトをJSON文字列に変換

```Plain
select json_string('{"Name": "Alice"}');
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```

例 2: PARSE_JSONの結果をJSON文字列に変換

```Plain
select json_string(parse_json('{"Name": "Alice"}'));
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```
