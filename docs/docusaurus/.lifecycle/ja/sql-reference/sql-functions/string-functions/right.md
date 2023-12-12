---
displayed_sidebar: "Japanese"
---

# 右

## 説明

この関数は、指定された文字列から指定された長さの文字を取得します。長さの単位：utf8 文字。
注意：この関数は[strright](strright.md)としても名前が付けられています。

## 構文

```SQL
VARCHAR right(VARCHAR str,INT len)
```

## 例

```SQL
MySQL > select right("Hello starrocks",9);
+-----------------------------+
| right('Hello starrocks', 9) |
+-----------------------------+
| starrocks                   |
+-----------------------------+
```

## キーワード

RIGHT