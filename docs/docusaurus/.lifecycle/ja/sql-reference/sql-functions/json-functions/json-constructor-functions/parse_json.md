---
displayed_sidebar: "Japanese"
---

# parse_json

## 説明

文字列をJSON値に変換します。

## 構文

```Haskell
parse_json(string_expr)
```

## パラメーター

`string_expr`： 文字列を表す式。STRING、VARCHAR、およびCHARデータ型のみがサポートされています。

## 戻り値

JSON値を返します。

> 注：文字列を標準のJSON値にパースできない場合、PARSE_JSON関数は`NULL`を返します（Example 5を参照）。JSON仕様についての情報は、[RFC 7159](https://tools.ietf.org/html/rfc7159?spm=a2c63.p38356.0.0.14d26b9fcp7fcf#page-4)を参照してください。

## 例

Example 1: `1`というSTRING値をJSON値の`1`に変換します。

```plaintext
mysql> SELECT parse_json('1');
+-----------------+
| parse_json('1') |
+-----------------+
| "1"             |
+-----------------+
```

Example 2: STRINGデータ型の配列をJSON配列に変換します。

```plaintext
mysql> SELECT parse_json('[1,2,3]');
+-----------------------+
| parse_json('[1,2,3]') |
+-----------------------+
| [1, 2, 3]             |
+-----------------------+ 
```

Example 3: STRINGデータ型のオブジェクトをJSONオブジェクトに変換します。

```plaintext
mysql> SELECT parse_json('{"star": "rocks"}');
+---------------------------------+
| parse_json('{"star": "rocks"}') |
+---------------------------------+
| {"star": "rocks"}               |
+---------------------------------+
```

Example 4: `NULL`のJSON値を構築します。

```plaintext
mysql> SELECT parse_json('null');
+--------------------+
| parse_json('null') |
+--------------------+
| "null"             |
+--------------------+
```

Example 5: 文字列を標準のJSON値にパースできない場合、PARSE_JSON関数は`NULL`を返します。この例では、`star`が二重引用符（"）で囲まれていないため、PARSE_JSON関数は`NULL`を返します。

```plaintext
mysql> SELECT parse_json('{star: "rocks"}');
+-------------------------------+
| parse_json('{star: "rocks"}') |
+-------------------------------+
| NULL                          |
+-------------------------------+
```

## キーワード

parse_json, parse json