---
displayed_sidebar: "Japanese"
---

# locate

## 説明

この関数は、文字列内の部分文字列の位置を見つけるために使用されます（1から数えて文字単位で測定）。第3引数のposが指定されている場合、それ以下の文字列内のsubstrの位置を検索し始めます。substrが見つからない場合、0を返します。

## 構文

```Haskell
INT locate(VARCHAR substr, VARCHAR str[, INT pos])
```

## 例

```Plain Text
MySQL > SELECT LOCATE('bar', 'foobarbar');
+----------------------------+
| locate('bar', 'foobarbar') |
+----------------------------+
|                          4 |
+----------------------------+

MySQL > SELECT LOCATE('xbar', 'foobar');
+--------------------------+
| locate('xbar', 'foobar') |
+--------------------------+
|                        0 |
+--------------------------+

MySQL > SELECT LOCATE('bar', 'foobarbar', 5);
+-------------------------------+
| locate('bar', 'foobarbar', 5) |
+-------------------------------+
|                             7 |
+-------------------------------+
```

## キーワード

LOCATE