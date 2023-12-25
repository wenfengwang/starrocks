---
displayed_sidebar: English
---

# ltrim

## 説明

`str` 引数の先頭（左側）から先頭のスペースまたは指定された文字を削除します。指定された文字の削除はStarRocks 2.5.0からサポートされています。

## 構文

```Haskell
VARCHAR ltrim(VARCHAR str[, VARCHAR characters])
```

## パラメーター

`str`: 必須。トリムする文字列で、VARCHAR値として評価されなければなりません。

`characters`: 省略可能。削除する文字で、VARCHAR値でなければなりません。このパラメータが指定されていない場合、デフォルトで文字列からスペースが削除されます。このパラメータを空文字列に設定すると、エラーが返されます。

## 戻り値

VARCHAR値を返します。

## 例

例1: 文字列の先頭からスペースを削除します。

```Plain Text
MySQL > SELECT ltrim('   ab d');
+------------------+
| ltrim('   ab d') |
+------------------+
| ab d             |
+------------------+
```

例2: 文字列の先頭から指定された文字を削除します。

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
