---
displayed_sidebar: Chinese
---

# json_string

## 機能

JSON 型を JSON 文字列に変換します。

## 文法

```SQL
json_string(json_object_expr)
```

## パラメータ説明

- `json_object_expr`：JSON オブジェクトの式で、JSON 型の列や、PARSE_JSON() などの JSON 関数で構築された JSON オブジェクトが使用できます。

## 戻り値の説明

VARCHAR 型の値を返します。

## 例

例1: JSON オブジェクトを JSON 文字列に変換します。

```Plain
select json_string('{"Name": "Alice"}');
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```

例2: PARSE_JSON() の結果を JSON 文字列に変換します。

```Plain
select json_string(parse_json('{"Name": "Alice"}'));
+----------------------------------+
| json_string('{"Name": "Alice"}') |
+----------------------------------+
| {"Name": "Alice"}                |
+----------------------------------+
```
