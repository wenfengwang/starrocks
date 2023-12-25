---
displayed_sidebar: English
---

# right

## 説明

この関数は、与えられた文字列の右側から指定された長さの文字列を返します。長さの単位: utf8 文字。
注: この関数は[strright](strright.md)としても知られています。

## 構文

```SQL
VARCHAR right(VARCHAR str, INT len)
```

## 例

```SQL
MySQL > SELECT right("Hello starrocks", 9);
+-----------------------------+
| right('Hello starrocks', 9) |
+-----------------------------+
| starrocks                   |
+-----------------------------+
```

## キーワード

RIGHT
