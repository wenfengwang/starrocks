---
displayed_sidebar: Chinese
---

# regexp_replace

## 機能

文字列 str に正規表現 pattern を適用し、一致した部分を repl で置換します。

### 文法

```Haskell
regexp_replace(str, pattern, repl)
```

## パラメータ説明

`str`: 対応するデータ型は VARCHAR です。

`pattern`: 対応するデータ型は VARCHAR です。

`repl`: 対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

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
