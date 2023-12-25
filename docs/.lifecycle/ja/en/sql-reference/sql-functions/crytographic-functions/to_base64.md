---
displayed_sidebar: English
---

# to_base64

## 説明

文字列をBase64エンコードされた文字列に変換します。この関数は[from_base64](from_base64.md)の逆関数です。

## 構文

```Haskell
to_base64(str);
```

## パラメーター

`str`: エンコードする文字列。VARCHAR型でなければなりません。

## 戻り値

VARCHAR型の値を返します。入力がNULLの場合はNULLが返されます。入力が空の場合はエラーが返されます。

この関数は1つの文字列のみを受け入れます。複数の入力文字列を指定するとエラーが発生します。

## 例

```Plain Text
mysql> select to_base64("starrocks");
+------------------------+
| to_base64('starrocks') |
+------------------------+
| c3RhcnJvY2tz           |
+------------------------+
1行がセットされました (0.00秒)

mysql> select to_base64(123);
+----------------+
| to_base64(123) |
+----------------+
| MTIz           |
+----------------+
```
