---
displayed_sidebar: English
---

# find_in_set

## 説明

この関数は、strlist内の最初のstrの位置を返します（1から数え始めます）。Strlistはコンマで区切られた文字列です。strが見つからない場合は0を返します。引数がNULLの場合、結果もNULLになります。

## 構文

```Haskell
INT find_in_set(VARCHAR str, VARCHAR strlist)
```

## 例

```Plain Text
MySQL > select find_in_set("b", "a,b,c");
+---------------------------+
| find_in_set('b', 'a,b,c') |
+---------------------------+
|                         2 |
+---------------------------+
```

## キーワード

FIND_IN_SET, FIND, IN, SET
