---
displayed_sidebar: English
---

# left

## 説明

この関数は、指定された文字列の左側から指定された数の文字を返します。長さの単位は utf8 文字です。
注: この関数は [strleft](strleft.md) としても知られています。

## 構文

```SQL
VARCHAR left(VARCHAR str, INT len)
```

## 例

```SQL
MySQL > select left("Hello starrocks", 5);
+----------------------------+
| left('Hello starrocks', 5) |
+----------------------------+
| Hello                      |
+----------------------------+
```

## キーワード

LEFT
