---
displayed_sidebar: "Japanese"
---

# hour

## 説明

指定された日付の時間を返します。返り値の範囲は0から23までです。

`date`パラメータはDATE型またはDATETIME型でなければなりません。

## 構文

```Haskell
INT HOUR(DATETIME|DATE date)
```

## 例

```Plain Text
MySQL > select hour('2018-12-31 23:59:59');
+-----------------------------+
| hour('2018-12-31 23:59:59') |
+-----------------------------+
|                          23 |
+-----------------------------+
```

## キーワード

HOUR