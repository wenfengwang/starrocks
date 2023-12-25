---
displayed_sidebar: English
---

# replace

## 説明

文字列内のすべての出現を別の文字列で置き換えます。この関数は`pattern`を検索する際に大文字と小文字を区別してマッチします。

この関数はv3.0からサポートされています。

注: 3.0より前では、この関数は[regexp_replace](../like-predicate-functions/regexp_replace.md)として実装されていました。

## 構文

```SQL
VARCHAR replace(VARCHAR str, VARCHAR pattern, VARCHAR repl)
```

## パラメータ

- `str`: 元の文字列です。

- `pattern`: 置き換える文字です。これは正規表現ではありません。

- `repl`: `pattern`の文字を置き換えるために使用される文字列です。

## 戻り値

指定された文字が置き換えられた文字列を返します。

引数のいずれかがNULLの場合、結果はNULLになります。

一致する文字が見つからない場合、元の文字列が返されます。

## 例

```plain
-- Replace '.' in 'a.b.c' with '+'.

MySQL > SELECT replace('a.b.c', '.', '+');
+----------------------------+
| replace('a.b.c', '.', '+') |
+----------------------------+
| a+b+c                      |
+----------------------------+

-- 一致する文字が見つからず、元の文字列が返されます。

MySQL > SELECT replace('a b c', '', '*');
+----------------------------+
| replace('a b c', '', '*')  |
+----------------------------+
| a b c                      |
+----------------------------+

-- 'like'を空の文字列で置き換えます。

MySQL > SELECT replace('We like StarRocks', 'like', '');
+------------------------------------------+
| replace('We like StarRocks', 'like', '') |
+------------------------------------------+
| We  StarRocks                            |
+------------------------------------------+

-- 一致する文字が見つからず、元の文字列が返されます。

MySQL > SELECT replace('He is awesome', 'handsome', 'smart');
+-----------------------------------------------+
| replace('He is awesome', 'handsome', 'smart') |
+-----------------------------------------------+
| He is awesome                                 |
+-----------------------------------------------+
```

## キーワード

REPLACE, replace
