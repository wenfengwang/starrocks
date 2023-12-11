---
displayed_sidebar: "Japanese"
---

# monthname（月名）

## 説明

指定された日付の月の名前を返します。

`date`パラメータはDATE型またはDATETIME型でなければなりません。

## 構文

```Haskell
VARCHAR MONTHNAME(date)
```

## 例

```Plain Text
MySQL > select monthname('2008-02-03 00:00:00');
+----------------------------------+
| monthname('2008-02-03 00:00:00') |
+----------------------------------+
| February                         |
+----------------------------------+
```

## キーワード

MONTHNAME, monthname