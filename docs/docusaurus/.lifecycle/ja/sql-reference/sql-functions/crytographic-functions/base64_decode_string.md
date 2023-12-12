---
displayed_sidebar: "Japanese"
---

# base64_decode_string

## 説明

この関数は、[from_base64](from_base64.md) と同じです。
Base64でエンコードされた文字列をデコードします。この関数は、[to_base64](to_base64.md) の逆です。

この関数は v3.0 からサポートされています。

## 構文

```Haskell
base64_decode_string(str);
```

## パラメータ

`str`: デコードする文字列です。VARCHARタイプである必要があります。

## 返り値

VARCHAR タイプの値を返します。入力が NULL または無効なBase64文字列の場合、NULL が返されます。入力が空の場合は、エラーが返されます。

この関数は1つの文字列のみを受け付けます。複数の入力文字列があると、エラーが発生します。

## 例

```Plain Text

mysql> select base64_decode_string(to_base64("Hello StarRocks"));
+----------------------------------------------------+
| base64_decode_string(to_base64('Hello StarRocks')) |
+----------------------------------------------------+
| Hello StarRocks                                    |
+----------------------------------------------------+
```