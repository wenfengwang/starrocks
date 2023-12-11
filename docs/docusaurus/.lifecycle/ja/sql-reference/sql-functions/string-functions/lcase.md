---
displayed_sidebar: "Japanese"
---

# lcase

## 説明

この関数は、文字列を小文字に変換します。lower 関数に類似しています。

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