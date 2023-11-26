---
displayed_sidebar: "Japanese"
---

# ucase

## 説明

この関数は文字列を大文字に変換します。関数upperと同様です。

## 構文

```Haskell
VARCHAR ucase(VARCHAR str)
```

## 例

```Plain Text
mysql> SELECT ucase("AbC123");
+-----------------+
|ucase('AbC123')  |
+-----------------+
|ABC123           |
+-----------------+
```

## キーワード

UCASE
