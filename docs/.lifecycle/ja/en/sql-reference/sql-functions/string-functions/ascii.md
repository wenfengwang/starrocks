---
displayed_sidebar: English
---

# ascii

## 説明

この関数は、指定された文字列の最も左の文字のASCII値を返します。

## 構文

```Haskell
INT ascii(VARCHAR str)
```

## 例

```Plain Text
MySQL > select ascii('1');
+------------+
| ascii('1') |
+------------+
|         49 |
+------------+

MySQL > select ascii('234');
+--------------+
| ascii('234') |
+--------------+
|           50 |
+--------------+
```

## キーワード

ASCII
