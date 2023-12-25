---
displayed_sidebar: English
---

# base64_decode_string

## 説明

この関数は [from_base64](from_base64.md) と同様です。
Base64エンコードされた文字列をデコードし、[to_base64](to_base64.md) の逆操作を行います。

この関数はv3.0からサポートされています。

## 構文

```Haskell
base64_decode_string(str);
```

## パラメーター

`str`: デコードする対象の文字列です。VARCHAR型である必要があります。

## 戻り値

VARCHAR型の値を返します。入力がNULLまたは無効なBase64文字列の場合はNULLが返されます。入力が空の場合はエラーが返されます。

この関数は一つの文字列のみを受け付けます。複数の入力文字列を指定するとエラーが発生します。

## 例

```Plain Text

mysql> select base64_decode_string(to_base64("Hello StarRocks"));
+----------------------------------------------------+
| base64_decode_string(to_base64('Hello StarRocks')) |
+----------------------------------------------------+
| Hello StarRocks                                    |
+----------------------------------------------------+
```
