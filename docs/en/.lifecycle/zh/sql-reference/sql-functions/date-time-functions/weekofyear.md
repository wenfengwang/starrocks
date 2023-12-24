---
displayed_sidebar: English
---

# WeekofYear

## 描述

返回一年内特定日期的周数。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT WEEKOFYEAR(DATETIME date)
```

## 例子

```Plain Text
MySQL > select weekofyear('2008-02-20 00:00:00');
+-----------------------------------+
| weekofyear('2008-02-20 00:00:00') |
+-----------------------------------+
|                                 8 |
+-----------------------------------+
```

## 关键词

WEEKOFYEAR