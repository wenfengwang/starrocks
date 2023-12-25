---
displayed_sidebar: English
---

# MONTH

## 説明

指定された日付の月を返します。戻り値は1から12の範囲です。

`date` パラメータは DATE 型または DATETIME 型でなければなりません。

## 構文

```Haskell
INT MONTH(DATETIME date)
```

## 例

```Plain Text
MySQL > select month('1987-01-01');
+-----------------------------+
|month('1987-01-01 00:00:00') |
+-----------------------------+
|                           1 |
+-----------------------------+
```

## キーワード

MONTH
