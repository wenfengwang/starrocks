---
displayed_sidebar: "Japanese"
---

# rtrim

## 説明

`str`引数の末尾（右側）から、末尾のスペースまたは指定された文字を削除します。指定された文字を削除する機能は、StarRocks 2.5.0 からサポートされています。

## 構文

```Haskell
VARCHAR rtrim(VARCHAR str[, VARCHAR characters]);
```

## パラメータ

`str`: 必須、トリムする文字列。これはVARCHAR値である必要があります。

`characters`: オプション、削除する文字。これはVARCHAR値である必要があります。このパラメータが指定されていない場合、デフォルトで文字列からスペースが削除されます。このパラメータが空の文字列に設定されている場合、エラーが返されます。

## 戻り値

VARCHAR値を返します。

## 例

Example 1: 文字列から末尾の3つのスペースを削除する。

```Plain Text
mysql> SELECT rtrim('   ab d   ');
+---------------------+
| rtrim('   ab d   ') |
+---------------------+
|    ab d             |
+---------------------+
1 行が返されました (0.00 秒)
```

Example 2: 文字列の末尾から指定された文字を削除する。

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