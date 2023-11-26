---
displayed_sidebar: "Japanese"
---

# lcase

## 説明

この関数は文字列を小文字に変換します。関数lowerと同様の機能です。

## 構文

```Haskell
VARCHAR lcase(VARCHAR str)
```

## 例

```Plain Text
mysql> SELECT lcase("AbC123");
+-----------------+
|lcase('AbC123')  |
+-----------------+
|abc123           |
+-----------------+
```

## キーワード

LCASE
