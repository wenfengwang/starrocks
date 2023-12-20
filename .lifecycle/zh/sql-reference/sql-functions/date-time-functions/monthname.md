---
displayed_sidebar: English
---

# 月份名稱

## 描述

返回指定日期的月份名稱。

日期參數必須是 DATE 或 DATETIME 類型。

## 語法

```Haskell
VARCHAR MONTHNAME(date)
```

## 範例

```Plain
MySQL > select monthname('2008-02-03 00:00:00');
+----------------------------------+
| monthname('2008-02-03 00:00:00') |
+----------------------------------+
| February                         |
+----------------------------------+
```

## 關鍵字

MONTHNAME，月份名稱
