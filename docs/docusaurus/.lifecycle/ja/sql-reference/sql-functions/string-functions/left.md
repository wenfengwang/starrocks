---
displayed_sidebar: "Japanese"
---

# 左

## 説明

この関数は、与えられた文字列の左側から指定された文字数の文字を返します。長さの単位：utf8文字。
注意：この関数は [strleft](strleft.md) としても知られています。

## 構文

```SQL
VARCHAR left(VARCHAR str,INT len)
```

## 例

```SQL
MySQL > select left("Hello starrocks",5);
+----------------------------+
| left('Hello starrocks', 5) |
+----------------------------+
| Hello                      |
+----------------------------+
```

## キーワード

LEFT