---
displayed_sidebar: "English"
---

# adddate,days_add

## Description

Adds a specified time interval to a date.

## Syntax

```Haskell
DATETIME ADDDATE(DATETIME|DATE date,INTERVAL expr type)
```

## Parameters

- `date`: 必须是有效的日期或日期时间表达式。
- `expr`: 您想要添加的时间间隔。必须是INT类型。
- `type`: 时间间隔的单位。只能设置为以下值之一：YEAR、MONTH、DAY、HOUR、MINUTE、SECOND。

## Return value

返回一个DATETIME值。如果日期不存在，例如`2020-02-30`，则返回NULL。如果日期是DATE值，则会转换为DATETIME值。

## Examples

```Plain Text
select adddate('2010-11-30 23:59:59', INTERVAL 2 DAY);
+-------------------------------------------------+
| adddate('2010-11-30 23:59:59', INTERVAL 2 DAY) |
+-------------------------------------------------+
| 2010-12-02 23:59:59                             |
+-------------------------------------------------+

select adddate('2010-12-03', INTERVAL 2 DAY);
+----------------------------------------+
| adddate('2010-12-03', INTERVAL 2 DAY) |
+----------------------------------------+
| 2010-12-05 00:00:00                    |
+----------------------------------------+
```