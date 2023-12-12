---
displayed_sidebar: "Japanese"
---

# dayofweek_iso（曜日ISO）

## 説明

指定された日付のISO標準曜日を`1`から`7`の範囲の整数で返します。この標準では、`1`は月曜日を表し、`7`は日曜日を表します。

## 構文

```Haskell
INT DAY_OF_WEEK_ISO(DATETIME date)
```

## パラメーター

`date`: 変換したい日付。DATEまたはDATETIME型である必要があります。

## 例

次の例は、日付`2023-01-01`のISO標準曜日を返します。

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