---
displayed_sidebar: English
---

# rtrim

## 説明

`str` 引数の末尾（右側）からトレーリングスペースまたは指定された文字を削除します。指定された文字を削除する機能はStarRocks 2.5.0からサポートされています。

## 構文

```Haskell
VARCHAR rtrim(VARCHAR str[, VARCHAR characters]);
```

## パラメーター

`str`: 必須。トリムする文字列で、VARCHAR値として評価される必要があります。

`characters`: 省略可能。削除する文字で、VARCHAR値である必要があります。このパラメータが指定されていない場合、デフォルトで文字列からスペースが削除されます。このパラメータを空文字列に設定すると、エラーが返されます。

## 戻り値

VARCHAR値を返します。

## 例

例 1: 文字列の末尾から3つのスペースを削除します。

```Plain Text
mysql> SELECT rtrim('   ab d   ');
+---------------------+
| rtrim('   ab d   ') |
+---------------------+
|    ab d             |
+---------------------+
1 row in set (0.00 sec)
```

例 2: 文字列の末尾から指定された文字を削除します。

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
