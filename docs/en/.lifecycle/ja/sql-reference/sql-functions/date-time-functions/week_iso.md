---
displayed_sidebar: "Japanese"
---

# week_iso

## 説明

指定された年の日付に対して、ISO標準の週番号を返します。

dateパラメータはDATEまたはDATETIME型である必要があります。

## 構文

```Haskell
INT WEEK_ISO(DATETIME date)
```

## 例

```Plain Text
MySQL > select week_iso('2008-02-20 00:00:00');
+-----------------------------------+
| week_iso('2008-02-20 00:00:00')   |
+-----------------------------------+
|                                 8 |
+-----------------------------------+
```

## キーワード

WEEK_ISO
