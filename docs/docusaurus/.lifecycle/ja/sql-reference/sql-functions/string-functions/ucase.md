---
displayed_sidebar: "Japanese"
---

# ucase

## Description

この関数は文字列を大文字に変換します。これは関数upperに類似しています。

## Syntax

```Haskell
VARCHAR ucase(VARCHAR str)
```

## Examples

```Plain Text
mysql> SELECT ucase("AbC123");
+-----------------+
|ucase('AbC123')  |
+-----------------+
|ABC123           |
+-----------------+
```

## keyword

UCASE