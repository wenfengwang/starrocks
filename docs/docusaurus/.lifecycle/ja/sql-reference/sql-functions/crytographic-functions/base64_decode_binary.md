```markdown
---
displayed_sidebar: "Japanese"
---

# base64_decode_binary

## Description

Base64でエンコードされた文字列をデコードしてBINARYを返します。

この関数はv3.0からサポートされています。

## 構文

```Haskell
base64_decode_binary(str);
```

## パラメータ

`str`: デコードする文字列。VARCHARタイプである必要があります。

## 返り値

VARBINARYタイプの値を返します。入力がNULLまたは無効なBase64文字列の場合はNULLが返されます。入力が空の場合、エラーが返されます。

この関数は1つの文字列のみを受け入れます。複数の入力文字列があるとエラーが発生します。

## 例

```Plain Text
mysql> select hex(base64_decode_binary(to_base64("Hello StarRocks")));
+---------------------------------------------------------+
| hex(base64_decode_binary(to_base64('Hello StarRocks'))) |
+---------------------------------------------------------+
| 48656C6C6F2053746172526F636B73                          |
+---------------------------------------------------------+

mysql> select base64_decode_binary(NULL);
+--------------------------------------------------------+
| base64_decode_binary(NULL)                             |
+--------------------------------------------------------+
| NULL                                                   |
+--------------------------------------------------------+
```