---
displayed_sidebar: English
---

# trim

## 説明

連続するスペースまたは指定された文字を引数 `str` の先頭と末尾から削除します。指定された文字の削除はStarRocks 2.5.0からサポートされています。

## 構文

```Haskell
VARCHAR trim(VARCHAR str[, VARCHAR characters])
```

## パラメーター

`str`: 必須。トリムする文字列で、VARCHAR値に評価される必要があります。

`characters`: 省略可能。削除する文字で、VARCHAR値である必要があります。このパラメーターが指定されていない場合、デフォルトで文字列からスペースが削除されます。このパラメーターが空文字列に設定された場合、エラーが返されます。

## 戻り値

VARCHAR値を返します。

## 例

例 1: 文字列の先頭と末尾から5つのスペースを削除します。

```Plain Text
MySQL > SELECT trim("   ab c  ");
+-------------------+
| trim('   ab c  ') |
+-------------------+
| ab c              |
+-------------------+
1 row in set (0.00 sec)
```

例 2: 文字列の先頭と末尾から指定された文字を削除します。

```Plain Text
MySQL > SELECT trim("abcd", "ad");
+--------------------+
| trim('abcd', 'ad') |
+--------------------+
| bc                 |
+--------------------+

MySQL > SELECT trim("xxabcdxx", "x");
+-----------------------+
| trim('xxabcdxx', 'x') |
+-----------------------+
| abcd                  |
+-----------------------+
```

## 参照

- [ltrim](ltrim.md)
- [rtrim](rtrim.md)
