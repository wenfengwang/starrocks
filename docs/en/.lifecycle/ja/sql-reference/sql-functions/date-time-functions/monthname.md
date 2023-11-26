---
displayed_sidebar: "Japanese"
---

# monthname

## 説明

指定された日付の月の名前を返します。

`date` パラメータは DATE 型または DATETIME 型である必要があります。

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
| 2月                             |
+----------------------------------+
```

## キーワード

MONTHNAME, monthname
