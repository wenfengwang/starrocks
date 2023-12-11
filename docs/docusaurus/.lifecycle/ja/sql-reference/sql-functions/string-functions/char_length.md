---
displayed_sidebar: "Japanese"
---

# char_length

## Description

この関数は文字列の長さを返します。マルチバイト文字の場合、文字の数を返します。現在、utf8コーディングのみをサポートしています。注意：この関数はcharacter_lengthとしても名前が付けられています。

## Syntax

```Haskell
INT char_length(VARCHAR str)
```

## Examples

```Plain Text
MySQL > select char_length("abc");
+--------------------+
| char_length('abc') |
+--------------------+
|                  3 |
+--------------------+
```

## keyword

CHAR_LENGTH, CHARACTER_LENGTH