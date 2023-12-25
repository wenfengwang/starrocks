---
displayed_sidebar: English
---

# parse_json

## 説明

文字列をJSON値に変換します。

## 構文

```Haskell
parse_json(string_expr)
```

## パラメーター

`string_expr`: 文字列を表す式です。STRING、VARCHAR、CHARデータ型のみがサポートされています。

## 戻り値

JSON値を返します。

> 注: 文字列が標準のJSON値に解析できない場合、`parse_json`関数は`NULL`を返します（例5を参照）。JSON仕様については、[RFC 7159](https://tools.ietf.org/html/rfc7159)を参照してください。

## 例

例1: STRING型の`1`をJSON値の`1`に変換します。

```plaintext
mysql> SELECT parse_json('1');
+-----------------+
| parse_json('1') |
+-----------------+
| 1               |
+-----------------+
```

例2: STRING型の配列をJSON配列に変換します。

```plaintext
mysql> SELECT parse_json('[1,2,3]');
+-----------------------+
| parse_json('[1,2,3]') |
+-----------------------+
| [1, 2, 3]             |
+-----------------------+ 
```

例3: STRING型のオブジェクトをJSONオブジェクトに変換します。

```plaintext
mysql> SELECT parse_json('{"star": "rocks"}');
+---------------------------------+
| parse_json('{"star": "rocks"}') |
+---------------------------------+
| {"star": "rocks"}               |
+---------------------------------+
```

例4: `NULL`のJSON値を構築します。

```plaintext
mysql> SELECT parse_json('null');
+--------------------+
| parse_json('null') |
+--------------------+
| null               |
+--------------------+
```

例5: 文字列が標準のJSON値に解析できない場合、`parse_json`関数は`NULL`を返します。この例では、`star`が二重引用符（"）で囲まれていません。したがって、`parse_json`関数は`NULL`を返します。

```plaintext
mysql> SELECT parse_json('{star: "rocks"}');
+-------------------------------+
| parse_json('{star: "rocks"}') |
+-------------------------------+
| NULL                          |
+-------------------------------+
```

## キーワード

parse_json, JSONを解析
