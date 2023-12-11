---
displayed_sidebar: "Japanese"
---

# dayofweek_iso

## 説明

指定された日付のISO標準曜日を `1` から `7` の範囲内の整数として返します。この標準では、`1` が月曜日を表し、`7` が日曜日を表します。

## 構文

```Haskell
INT DAY_OF_WEEK_ISO(DATETIME date)
```

## パラメータ

`date`: 変換したい日付。DATEまたはDATETIME型でなければなりません。

## 例

次の例は、日付 `2023-01-01` のISO標準曜日を返します:

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