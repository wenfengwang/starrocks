---
displayed_sidebar: "Japanese"
---

# concat

## 説明

この関数は複数の文字列を結合します。パラメータのいずれかがNULLの場合、NULLを返します。

## 構文

```Haskell
VARCHAR concat(VARCHAR,...)
```

## 例

```Plain Text
MySQL > select concat("a", "b");
+------------------+
| concat('a', 'b') |
+------------------+
| ab               |
+------------------+

MySQL > select concat("a", "b", "c");
+-----------------------+
| concat('a', 'b', 'c') |
+-----------------------+
| abc                   |
+-----------------------+

MySQL > select concat("a", null, "c");
+------------------------+
| concat('a', NULL, 'c') |
+------------------------+
| NULL                   |
+------------------------+
```

## キーワード

CONCAT
