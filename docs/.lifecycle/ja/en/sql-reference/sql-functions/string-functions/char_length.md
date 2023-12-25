---
displayed_sidebar: English
---

# char_length

## 説明

この関数は文字列の長さを返します。マルチバイト文字に対しては、文字数を返します。現在、utf8コーディングのみをサポートしています。注: この関数は character_length としても知られています。

## 構文

```Haskell
INT char_length(VARCHAR str)
```

## 例

```Plain Text
MySQL > select char_length("abc");
+--------------------+
| char_length('abc') |
+--------------------+
|                  3 |
+--------------------+
```

## キーワード

CHAR_LENGTH, CHARACTER_LENGTH
