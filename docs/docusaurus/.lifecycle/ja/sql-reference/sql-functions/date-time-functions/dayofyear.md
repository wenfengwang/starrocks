---
displayed_sidebar: "Japanese"
---

# dayofyear

## Description

指定された日付の年内日数を返します。

`date`パラメータはDATEまたはDATETIMEタイプである必要があります。

## Syntax

```Haskell
INT DAYOFYEAR(DATETIME|DATE date)
```

## Examples

```Plain Text
MySQL > select dayofyear('2007-02-03 00:00:00');
+----------------------------------+
| dayofyear('2007-02-03 00:00:00') |
+----------------------------------+
|                               34 |
+----------------------------------+
```

## keyword

DAYOFYEAR