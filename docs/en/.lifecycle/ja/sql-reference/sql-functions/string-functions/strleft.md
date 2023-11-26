---
displayed_sidebar: "Japanese"
---

# strleft

## 説明

この関数は、指定された長さ（左から）の文字列から指定された数の文字を抽出します。長さの単位：UTF8文字。
注意：この関数は[left](left.md)としても呼ばれます。

## 構文

```SQL
VARCHAR strleft(VARCHAR str,INT len)
```

## 例

```SQL
MySQL > select strleft("Hello starrocks",5);
+-------------------------------+
| strleft('Hello starrocks', 5) |
+-------------------------------+
| Hello                         |
+-------------------------------+
```

## キーワード

STRLEFT
