---
displayed_sidebar: English
---

# strleft

## 説明

この関数は、指定された長さの文字列から(左から開始して)文字を抽出します。長さの単位はUTF-8文字です。
注: この関数は[left](left.md)としても知られています。

## 構文

```SQL
VARCHAR strleft(VARCHAR str, INT len)
```

## 例

```SQL
MySQL > SELECT strleft("Hello starrocks", 5);
+-------------------------------+
| strleft('Hello starrocks', 5) |
+-------------------------------+
| Hello                         |
+-------------------------------+
```

## キーワード

STRLEFT
