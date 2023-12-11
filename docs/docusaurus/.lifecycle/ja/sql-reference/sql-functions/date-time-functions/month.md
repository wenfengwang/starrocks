---
displayed_sidebar: "Japanese"
---

# 月

## 説明

指定された日付の月を返します。返り値は1から12の範囲です。

`date`パラメータはDATEもしくはDATETIMEの型でなければなりません。

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