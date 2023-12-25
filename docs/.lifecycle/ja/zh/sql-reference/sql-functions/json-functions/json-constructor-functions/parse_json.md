---
displayed_sidebar: Chinese
---

# parse_json

## 機能

文字列型のデータをJSON型のデータに構築します。

## 文法

```Plain Text
PARSE_JSON(string_expr)
```

## パラメータ説明

`string_expr`: 文字列の式。サポートされるデータ型は文字列型（STRING、VARCHAR、CHAR）です。

## 戻り値の説明

JSON型の値を返します。

> 文字列が標準のJSONに解析できない場合は、NULLを返します。例五を参照してください。JSONの標準については、[RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)を参照してください。

## 例

例一：文字列型の「1」をJSON型の「1」に構築します。

```Plain Text
mysql> SELECT PARSE_JSON('1');
+-----------------+
| parse_json('1') |
+-----------------+
| "1"             |
+-----------------+
```

例二：文字列型の配列をJSON型の配列に構築します。

```Plain Text
mysql> SELECT PARSE_JSON('[1,2,3]');
+-----------------------+
| parse_json('[1,2,3]') |
+-----------------------+
| [1, 2, 3]             |
+-----------------------+ 
```

例三：文字列型のオブジェクトをJSON型のオブジェクトに構築します。

```Plain Text
mysql> SELECT PARSE_JSON('{"star": "rocks"}');
+---------------------------------+
| parse_json('{"star": "rocks"}') |
+---------------------------------+
| {"star": "rocks"}               |
+---------------------------------+
```

例四：JSON型のNULLを構築します。

```Plain Text
mysql> SELECT PARSE_JSON('null');
+--------------------+
| parse_json('null') |
+--------------------+
| "null"             |
+--------------------+
```

例五：文字列が標準のJSONに解析できない場合は、NULLを返します。以下の例では、starがダブルクォートで囲まれていないため、有効なJSONとして解析できず、NULLが返されます。

```Plain Text
mysql> SELECT PARSE_JSON('{star: "rocks"}');
+-------------------------------+
| parse_json('{star: "rocks"}') |
+-------------------------------+
| NULL                          |
+-------------------------------+
```

## キーワード

parse_json, parse json
