---
displayed_sidebar: English
---

# TIMESTAMPADD

## 描述

向日期或日期时间表达式 `datetime_expr` 添加一个整数表达式间隔。

间隔的单位必须是以下之一：

SECOND, MINUTE, HOUR, DAY, WEEK, MONTH, 或 YEAR。

## 语法

```Haskell
DATETIME TIMESTAMPADD(unit, interval, DATETIME datetime_expr)
```

## 示例

```plain

MySQL > SELECT TIMESTAMPADD(MINUTE,1,'2019-01-02');
+------------------------------------------------+
| timestampadd(MINUTE, 1, '2019-01-02 00:00:00') |
+------------------------------------------------+
| 2019-01-02 00:01:00                            |
+------------------------------------------------+

MySQL > SELECT TIMESTAMPADD(WEEK,1,'2019-01-02');
+----------------------------------------------+
| timestampadd(WEEK, 1, '2019-01-02 00:00:00') |
+----------------------------------------------+
| 2019-01-09 00:00:00                          |
+----------------------------------------------+
```

## 关键字

TIMESTAMPADD