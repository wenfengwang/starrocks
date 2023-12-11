---
displayed_sidebar: "Japanese"
---

# regexp_replace（正規表現置換）

## 説明

この関数は、正規表現パターンに一致する文字列を置換するためにreplを使用します。

## 構文

```Haskell
VARCHAR regexp_replace(VARCHAR str, VARCHAR pattern, VARCHAR repl)
```

## 例

```Plain Text
MySQL > SELECT regexp_replace('a b c', " ", "-");
+-----------------------------------+
| regexp_replace('a b c', ' ', '-') |
+-----------------------------------+
| a-b-c                             |
+-----------------------------------+

MySQL > SELECT regexp_replace('a b c','(b)','<\\1>');
+----------------------------------------+
| regexp_replace('a b c', '(b)', '<\1>') |
+----------------------------------------+
| a <b> c                                |
+----------------------------------------+
```

## キーワード

REGEXP_REPLACE,REGEXP,REPLACE