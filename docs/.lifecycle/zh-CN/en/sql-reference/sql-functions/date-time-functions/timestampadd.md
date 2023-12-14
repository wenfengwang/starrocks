---
displayed_sidebar: "Chinese"
---

# timestampadd

## Description

将整数表达式间隔添加到日期或日期时间表达式' datetime_expr'。

如上所述，间隔的单位必须是以下之一：

SECOND，MINUTE，HOUR，DAY，WEEK，MONTH，或YEAR。

## Syntax

```Haskell
DATETIME TIMESTAMPADD(unit, interval, DATETIME datetime_expr)
```

## Examples

```plain text

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

## keyword

TIMESTAMPADD