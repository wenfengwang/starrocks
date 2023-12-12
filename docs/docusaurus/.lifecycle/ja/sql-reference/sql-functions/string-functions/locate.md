---
displayed_sidebar: "Japanese"
---

# locate

## 説明

この関数は、文字列内の部分文字列の位置を検索するために使用されます（1から数えて文字で測定されます）。3番目の引数posが指定されている場合、それ以下の位置から文字列内の部分文字列の位置を検出し始めます。strが見つからない場合、0を返します。

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