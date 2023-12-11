---
displayed_sidebar: "Japanese"
---

# strright

## 説明

この関数は、指定された長さから始まり、文字列から指定された文字数を抽出します（右側から）。長さの単位：utf-8 文字。
注意：この関数は [right](right.md) としても呼ばれます。

## 構文

```SQL
VARCHAR strright(VARCHAR str,INT len)
```

## 例

```SQL
MySQL > select strright("Hello starrocks",9);
+--------------------------------+
| strright('Hello starrocks', 9) |
+--------------------------------+
| starrocks                      |
+--------------------------------+
```

## キーワード

STRRIGHT