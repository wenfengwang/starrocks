---
displayed_sidebar: "Japanese"
---

# weekofyear

## Description

年内の指定された日付の週番号を返します。

`date` パラメーターは DATE 型または DATETIME 型である必要があります。

## Syntax

```Haskell
INT WEEKOFYEAR(DATETIME date)
```

## Examples

```Plain Text
MySQL > select weekofyear('2008-02-20 00:00:00');
+-----------------------------------+
| weekofyear('2008-02-20 00:00:00') |
+-----------------------------------+
|                                 8 |
+-----------------------------------+
```

## keyword

WEEKOFYEAR