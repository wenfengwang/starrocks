---
displayed_sidebar: Chinese
---

# base64_decode_string

## 機能

[from_base64()](../crytographic-functions/from_base64.md) 関数と同じく、Base64エンコードされた文字列をデコードするための関数で、[to_base64()](../crytographic-functions/to_base64.md) 関数の逆関数です。

この関数はバージョン3.0からサポートされています。

## 構文

```Haskell
VARCHAR base64_decode_string(VARCHAR str);
```

## パラメータ説明

`str`：デコードする文字列で、VARCHAR型でなければなりません。

## 戻り値の説明

VARCHAR型の値を返します。入力が `NULL` または無効なBase64エンコード文字列の場合は `NULL` を返します。入力が空の場合はエラーメッセージを返します。この関数は一つの文字列のみを入力としてサポートしており、複数の文字列を入力するとエラーになります。

## 例

```Plain
mysql> select base64_decode_string(to_base64("Hello StarRocks"));
+----------------------------------------------------+
| base64_decode_string(to_base64('Hello StarRocks')) |
+----------------------------------------------------+
| Hello StarRocks                                    |
+----------------------------------------------------+
```

## キーワード

BASE64_DECODE_STRING
