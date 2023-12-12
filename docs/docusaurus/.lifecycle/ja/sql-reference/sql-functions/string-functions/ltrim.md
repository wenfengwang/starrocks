---
displayed_sidebar: "Japanese"
---

# ltrim

## 説明

`str`引数の先頭（左側）から先頭のスペースまたは指定された文字を削除します。指定された文字の削除は、StarRocks 2.5.0からサポートされています。

## 構文

```Haskell
VARCHAR ltrim(VARCHAR str[, VARCHAR characters])
```

## パラメータ

`str`: 必須、切り捨てる文字列で、VARCHAR値に評価される必要があります。

`characters`: オプション、削除する文字列で、VARCHAR値である必要があります。このパラメータが指定されていない場合、デフォルトで文字列からスペースが削除されます。このパラメータが空の文字列に設定されている場合、エラーが返されます。

## 戻り値

VARCHAR値を返します。

## 例

例1：文字列の先頭からスペースを削除します。

```Plain Text
MySQL > SELECT ltrim('   ab d');
+------------------+
| ltrim('   ab d') |
+------------------+
| ab d             |
+------------------+
```

例2：文字列の先頭から指定された文字を削除します。

```Plain Text
MySQL > SELECT ltrim("xxabcdxx", "x");
+------------------------+
| ltrim('xxabcdxx', 'x') |
+------------------------+
| abcdxx                 |
+------------------------+
```

## 参照

- [trim](trim.md)
- [rtrim](rtrim.md)

## キーワード

LTRIM