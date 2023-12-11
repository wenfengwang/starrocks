---
displayed_sidebar: "Japanese"
---

# repeat（繰り返し）

## 説明

この関数は、`count`に従って`str`を繰り返します。`count`が1未満の場合、空の文字列を返します。`str`または`count`がNULLの場合、NULLを返します。

## 構文

```Haskell
VARCHAR repeat(VARCHAR str, INT count)
```

## 例

```Plain Text
MySQL > SELECT repeat("a", 3);
+----------------+
| repeat('a', 3) |
+----------------+
| aaa            |
+----------------+

MySQL > SELECT repeat("a", -1);
+-----------------+
| repeat('a', -1) |
+-----------------+
|                 |
+-----------------+
```

## キーワード

REPEAT