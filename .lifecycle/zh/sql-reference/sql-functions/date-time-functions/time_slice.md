---
displayed_sidebar: English
---

# 时间片段

## 描述

此功能可根据指定的时间粒度，将给定时间转换成时间间隔的起始或结束点。

该函数自v2.3版本起提供支持。v2.5版本起支持将给定时间转换为时间间隔的结束点。

## 语法

```Haskell
DATETIME time_slice(DATETIME dt, INTERVAL N type[, boundary])
```

## 参数

- dt：需转换的时间，数据类型为DATETIME。
- INTERVAL N 类型：时间粒度，例如，间隔5秒钟。
  - N 代表时间间隔的长度，必须是 INT（整数）类型。
  - type 指的是时间单位，可以是 YEAR（年）、QUARTER（季度）、MONTH（月）、WEEK（周）、DAY（日）、HOUR（小时）、MINUTE（分钟）、SECOND（秒）。
- boundary：可选项。用于指定返回时间间隔的开始（FLOOR）还是结束（CEIL）。有效值为：FLOOR（下限）、CEIL（上限）。若不指定此参数，默认值为 FLOOR。该参数自v2.5版本起得到支持。

## 返回值

返回的数据类型为 DATETIME。

## 使用说明

时间间隔的起点为公元0001年01月01日 00:00:00。

## 示例

以下示例基于 test_all_type_select 表。

```Plaintext
select * from test_all_type_select order by id_int;

+------------+---------------------+--------+
| id_date    | id_datetime         | id_int |
+------------+---------------------+--------+
| 2052-12-26 | 1691-12-23 04:01:09 |      0 |
| 2168-08-05 | 2169-12-18 15:44:31 |      1 |
| 1737-02-06 | 1840-11-23 13:09:50 |      2 |
| 2245-10-01 | 1751-03-21 00:19:04 |      3 |
| 1889-10-27 | 1861-09-12 13:28:18 |      4 |
+------------+---------------------+--------+
5 rows in set (0.06 sec)
```

示例 1：将给定的 DATETIME 值转换为5秒时间间隔的起始点，不指定boundary参数。

```Plaintext
select time_slice(id_datetime, interval 5 second)
from test_all_type_select
order by id_int;

+---------------------------------------------------+
| time_slice(id_datetime, INTERVAL 5 second, floor) |
+---------------------------------------------------+
| 1691-12-23 04:01:05                               |
| 2169-12-18 15:44:30                               |
| 1840-11-23 13:09:50                               |
| 1751-03-21 00:19:00                               |
| 1861-09-12 13:28:15                               |
+---------------------------------------------------+
5 rows in set (0.16 sec)
```

示例 2：将给定的 DATETIME 值转换为5天时间间隔的起始点，boundary参数设为 FLOOR。

```Plaintext
select time_slice(id_datetime, interval 5 day, FLOOR)
from test_all_type_select
order by id_int;

+------------------------------------------------+
| time_slice(id_datetime, INTERVAL 5 day, floor) |
+------------------------------------------------+
| 1691-12-22 00:00:00                            |
| 2169-12-16 00:00:00                            |
| 1840-11-21 00:00:00                            |
| 1751-03-18 00:00:00                            |
| 1861-09-12 00:00:00                            |
+------------------------------------------------+
5 rows in set (0.15 sec)
```

示例 3：将给定的 DATETIME 值转换为5天时间间隔的结束点。

```Plaintext
select time_slice(id_datetime, interval 5 day, CEIL)
from test_all_type_select
order by id_int;

+-----------------------------------------------+
| time_slice(id_datetime, INTERVAL 5 day, ceil) |
+-----------------------------------------------+
| 1691-12-27 00:00:00                           |
| 2169-12-21 00:00:00                           |
| 1840-11-26 00:00:00                           |
| 1751-03-23 00:00:00                           |
| 1861-09-17 00:00:00                           |
+-----------------------------------------------+
5 rows in set (0.12 sec)
```
