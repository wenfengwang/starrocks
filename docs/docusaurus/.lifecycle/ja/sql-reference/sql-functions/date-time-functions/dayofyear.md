---
displayed_sidebar: "Japanese"
---

# dayofyear

## 説明

指定された日付の年内の日数を返します。

`date`パラメータはDATEまたはDATETIME型でなければなりません。

## 構文

```Haskell
INT DAYOFYEAR(DATETIME|DATE date)
```

## 例

```Plain Text
MySQL > select dayofyear('2007-02-03 00:00:00');
+----------------------------------+
| dayofyear('2007-02-03 00:00:00') |
+----------------------------------+
|                               34 |
+----------------------------------+
```

## キーワード

DAYOFYEAR