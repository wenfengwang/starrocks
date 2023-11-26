---
displayed_sidebar: "Japanese"
---

# char_length

## 説明

この関数は文字列の長さを返します。マルチバイト文字の場合、文字数を返します。現在はutf8のみをサポートしています。注意：この関数はcharacter_lengthとしても呼ばれます。

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
