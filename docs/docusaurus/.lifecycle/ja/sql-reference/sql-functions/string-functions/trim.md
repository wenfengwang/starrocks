---
displayed_sidebar: "Japanese"
---

# trim

## 説明

`str` 引数から連続するスペースまたは指定された文字を削除します。指定された文字の削除は StarRocks 2.5.0 からサポートされています。

## 構文

```Haskell
VARCHAR trim(VARCHAR str[, VARCHAR characters])
```

## パラメータ

`str`: 必須、トリムする文字列、VARCHAR 値である必要があります。

`characters`: オプション、削除する文字列、VARCHAR 値である必要があります。このパラメータが指定されていない場合、デフォルトで文字列からスペースが削除されます。このパラメータが空の文字列に設定されている場合、エラーが返されます。

## 戻り値

VARCHAR 値を返します。

## 例

例1: 文字列の先頭と末尾から5つのスペースを除去します。

```Plain Text
MySQL > SELECT trim("   ab c  ");
+-------------------+
| trim('   ab c  ') |
+-------------------+
| ab c              |
+-------------------+
1 row in set (0.00 sec)
```

例2: 文字列の先頭と末尾から指定された文字を除去します。

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