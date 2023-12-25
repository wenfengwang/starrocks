---
displayed_sidebar: English
---

# from_base64

## 説明

Base64エンコードされた文字列をデコードします。この関数は[to_base64](to_base64.md)の逆関数です。

## 構文

```Haskell
from_base64(str);
```

## パラメーター

`str`: デコードする対象の文字列です。VARCHAR型である必要があります。

## 戻り値

VARCHAR型の値を返します。入力がNULLまたは無効なBase64文字列の場合はNULLが返されます。入力が空の場合はエラーが返されます。

この関数は一つの文字列のみを受け付けます。複数の入力文字列を指定するとエラーが発生します。

## 例

```Plain Text
mysql> select from_base64("starrocks");
+--------------------------+
| from_base64('starrocks') |
+--------------------------+
| ²֫®$                       |
+--------------------------+
1 row in set (0.00 sec)

mysql> select from_base64('c3RhcnJvY2tz');
+-----------------------------+
| from_base64('c3RhcnJvY2tz') |
+-----------------------------+
| starrocks                   |
+-----------------------------+
```
