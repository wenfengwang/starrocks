---
displayed_sidebar: "Japanese"
---

# dayofweek_iso

## 説明

指定された日付のISO標準の曜日番号を返します。

日付パラメータはDATEまたはDATETIME型である必要があります。

## 構文

```Haskell
INT DAY_OF_WEEK_ISO(DATETIME date)
```

## 例

```Plain Text
mysql> select dayofweek_iso("2023-01-01");
+-----------------------------+
| dayofweek_iso('2023-01-01') |
+-----------------------------+
|                           7 |
+-----------------------------+
```

## キーワード

DAY_OF_WEEK_ISO
