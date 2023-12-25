---
displayed_sidebar: English
---

# lower

## 説明

引数に含まれる全ての文字列を小文字に変換します。

## 構文

```Haskell
INT lower(VARCHAR str)
```

## 例

```Plain Text
MySQL > SELECT lower("AbC123");
+-----------------+
| lower('AbC123') |
+-----------------+
| abc123          |
+-----------------+
```

## キーワード

LOWER
