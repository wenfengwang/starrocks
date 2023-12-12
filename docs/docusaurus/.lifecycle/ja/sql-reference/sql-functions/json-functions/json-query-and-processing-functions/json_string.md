---
displayed_sidebar: "Japanese"
---

# json_string

## 説明

JSONオブジェクトをJSON文字列に変換する

## 構文

```SQL
json_string(json_object_expr)
```

## パラメータ

- `json_object_expr`: JSONオブジェクトを表す式。オブジェクトはJSON列であるか、PARSE_JSONなどのJSONコンストラクタ関数によって生成されたJSONオブジェクトである可能性があります。

## 戻り値

VARCHAR値を返します。

## 例

Example 1: JSONオブジェクトをJSON文字列に変換する

```Plain
select json_string('{"Name": "Alice"}');
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```

Example 2: PARSE_JSONの結果をJSON文字列に変換する

```Plain
select json_string(parse_json('{"Name": "Alice"}'));
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```