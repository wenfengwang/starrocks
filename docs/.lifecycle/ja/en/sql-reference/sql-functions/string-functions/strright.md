---
displayed_sidebar: English
---

# strright

## 説明

この関数は、指定された長さの文字列から（右から開始して）文字数を抽出します。長さの単位はUTF-8文字です。
注: この関数は[right](right.md)としても知られています。

## 構文

```SQL
VARCHAR strright(VARCHAR str, INT len)
```

## 例

```SQL
MySQL > SELECT strright("Hello starrocks", 9);
+--------------------------------+
| strright('Hello starrocks', 9) |
+--------------------------------+
| starrocks                      |
+--------------------------------+
```

## キーワード

STRRIGHT
