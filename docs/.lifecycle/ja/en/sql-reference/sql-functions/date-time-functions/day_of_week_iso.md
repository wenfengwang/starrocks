---
displayed_sidebar: English
---

# dayofweek_iso

## 説明

指定された日付に対するISO標準の曜日を、`1`から`7`の範囲の整数で返します。この標準では、`1`は月曜日を、`7`は日曜日を表します。

## 構文

```Haskell
INT DAY_OF_WEEK_ISO(DATETIME date)
```

## パラメーター

`date`: 変換したい日付です。DATE型またはDATETIME型でなければなりません。

## 例

以下の例は、日付`2023-01-01`に対するISO標準の曜日を返します。

```SQL
MySQL > select dayofweek_iso('2023-01-01');
+-----------------------------+
| dayofweek_iso('2023-01-01') |
+-----------------------------+
|                           7 |
+-----------------------------+
```

## キーワード

DAY_OF_WEEK_ISO
