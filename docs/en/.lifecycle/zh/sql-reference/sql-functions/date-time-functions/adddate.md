---
displayed_sidebar: English
---

# adddate,days_add

## 描述

向日期添加指定的时间间隔。

## 语法

```Haskell
DATETIME ADDDATE(DATETIME|DATE date,INTERVAL expr type)
```

## 参数

- `date`：必须是有效的日期或日期时间表达式。
- `expr`：要添加的时间间隔。必须为 INT 类型。
- `type`：时间间隔的单位。只能设置为以下任一值：YEAR、MONTH、DAY、HOUR、MINUTE、SECOND。

## 返回值

返回一个 DATETIME 值。如果日期不存在，例如 `2020-02-30`，则返回 NULL。如果日期是 DATE 值，则将其转换为 DATETIME 值。

## 例子

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