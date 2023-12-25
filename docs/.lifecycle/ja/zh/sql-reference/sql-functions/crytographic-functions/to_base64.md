---
displayed_sidebar: Chinese
---

# to_base64

## 機能

文字列 `str` を Base64 エンコードします。逆関数は [from_base64](from_base64.md) です。

## 文法

```Haskell
to_base64(str);
```

## パラメータ説明

`str`: エンコードする文字列。サポートされるデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。入力が NULL の場合は NULL を返します。入力が空の場合はエラーを返します。

この関数は一つの文字列のみを受け取ります。複数の文字列を入力するとエラーが返されます。

## 例

```Plain Text
mysql> select to_base64("starrocks");
+------------------------+
| to_base64('starrocks') |
+------------------------+
| c3RhcnJvY2tz           |
+------------------------+
1 row in set (0.00 sec)

mysql> select to_base64(123);
+----------------+
| to_base64(123) |
+----------------+
| MTIz           |
+----------------+
```
