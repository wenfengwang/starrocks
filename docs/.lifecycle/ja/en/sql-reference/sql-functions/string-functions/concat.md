---
displayed_sidebar: English
---

# concat

## 説明

この関数は、複数の文字列を結合します。パラメーター値のいずれかがNULLの場合は、NULLを返します。

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

MySQL > select concat("a", NULL, "c");
+------------------------+
| concat('a', NULL, 'c') |
+------------------------+
| NULL                   |
+------------------------+
```

## キーワード

CONCAT
