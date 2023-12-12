---
displayed_sidebar: "Japanese"
---

# replace

## 説明

文字列内のすべての文字の出現を別の文字列で置き換えます。この関数は、`pattern` を検索する際に大文字と小文字を区別します。

この機能は v3.0 からサポートされています。

注意: 3.0 より前は、この機能は [regexp_replace](../like-predicate-functions/regexp_replace.md) として実装されていました。

## 構文

```SQL
VARCHAR replace(VARCHAR str, VARCHAR pattern, VARCHAR repl)
```

## パラメータ

- `str`: 元の文字列。

- `pattern`: 置き換える文字。

- `repl`: `pattern` 内の文字を置き換えるために使用される文字列。

## 戻り値

指定した文字が置換された文字列が返されます。

いずれかの引数が NULL の場合、結果も NULL となります。

一致する文字が見つからない場合、元の文字列が返されます。

## 例

```plain
-- 'a.b.c' 内の '.' を '+' に置き換える。

MySQL > SELECT replace('a.b.c', '.', '+');
+----------------------------+
| replace('a.b.c', '.', '+') |
+----------------------------+
| a+b+c                      |
+----------------------------+

-- 一致する文字が見つからないため、元の文字列が返されます。

MySQL > SELECT replace('a b c', '', '*');
+----------------------------+
| replace('a b c', '', '*') |
+----------------------------+
| a b c                      |
+----------------------------+

-- 'like' を空の文字列に置き換える。

MySQL > SELECT replace('We like StarRocks', 'like', '');
+------------------------------------------+
| replace('We like StarRocks', 'like', '') |
+------------------------------------------+
| We  StarRocks                            |
+------------------------------------------+

-- 一致する文字が見つからないため、元の文字列が返されます。

MySQL > SELECT replace('He is awesome', 'handsome', 'smart');
+-----------------------------------------------+
| replace('He is awesome', 'handsome', 'smart') |
+-----------------------------------------------+
| He is awesome                                 |
+-----------------------------------------------+
```

## キーワード

REPLACE, replace