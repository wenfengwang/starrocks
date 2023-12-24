---
displayed_sidebar: English
---

# Dayofyear

## 描述

返回给定日期的年份中的某一天。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT DAYOFYEAR(DATETIME|DATE date)
```

## 例子

```Plain Text
MySQL > select dayofyear('2007-02-03 00:00:00');
+----------------------------------+
| dayofyear('2007-02-03 00:00:00') |
+----------------------------------+
|                               34 |
+----------------------------------+
```

## 关键词

DAYOFYEAR