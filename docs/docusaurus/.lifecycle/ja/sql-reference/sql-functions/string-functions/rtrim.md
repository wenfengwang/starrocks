---
displayed_sidebar: "Japanese"
---

# rtrim

## 説明

`str`引数の末尾（右側）から、末尾のスペースまたは指定された文字を削除します。指定された文字の削除は、StarRocks 2.5.0からサポートされています。

## 構文

```Haskell
VARCHAR rtrim(VARCHAR str[, VARCHAR characters]);
```

## パラメーター

`str`: 必須、trimする文字列、これはVARCHAR値に評価される必要があります。

`characters`: オプション、削除する文字、これはVARCHAR値でなければなりません。このパラメータが指定されていない場合、デフォルトで文字列からスペースが削除されます。このパラメータが空の文字列に設定されている場合、エラーが返されます。

## 戻り値

VARCHAR値を返します。

## 例

例1: 文字列から末尾の3つのスペースを削除します。

```Plain Text
mysql> SELECT rtrim('   ab d   ');
+---------------------+
| rtrim('   ab d   ') |
+---------------------+
|    ab d             |
+---------------------+
1 row in set (0.00 sec)
```

例2: 文字列の末尾から指定された文字を削除します。

```Plain Text
MySQL > SELECT rtrim("xxabcdxx", "x");
+------------------------+
| rtrim('xxabcdxx', 'x') |
+------------------------+
| xxabcd                 |
+------------------------+
```

## 参照

- [trim](trim.md)
- [ltrim](ltrim.md)