---
displayed_sidebar: English
---

# DAYNAME

## 説明

日付に対応する曜日を返します。

`date` パラメータは DATE 型または DATETIME 型でなければなりません。

## 構文

```Haskell
VARCHAR DAYNAME(date)
```

## 例

```Plain Text
MySQL > select DAYNAME('2007-02-03 00:00:00');
+--------------------------------+
| DAYNAME('2007-02-03 00:00:00') |
+--------------------------------+
| Saturday                       |
+--------------------------------+
```

## キーワード

DAYNAME
