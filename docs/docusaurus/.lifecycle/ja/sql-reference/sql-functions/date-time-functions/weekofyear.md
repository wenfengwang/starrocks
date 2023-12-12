---
displayed_sidebar: "Japanese"
---

# weekofyear

## 説明

年内の特定の日付に対する週番号を返します。

`date` パラメータは DATE 型または DATETIME 型である必要があります。

## 構文

```Haskell
INT WEEKOFYEAR(DATETIME date)
```

## 例

```Plain Text
MySQL > select weekofyear('2008-02-20 00:00:00');
+-----------------------------------+
| weekofyear('2008-02-20 00:00:00') |
+-----------------------------------+
|                                 8 |
+-----------------------------------+
```

## キーワード

WEEKOFYEAR