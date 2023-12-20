---
displayed_sidebar: English
---

# 日期切片

## 描述

根据指定的时间粒度，将给定时间转换为时间间隔的起始或结束点。

该函数自v2.5版本起支持。

## 语法

```Haskell
DATE date_slice(DATE dt, INTERVAL N type[, boundary])
```

## 参数

- dt：要转换的时间，DATE类型。
- INTERVAL N 类型：时间粒度，例如，间隔5天。
  - N代表时间间隔的长度。它必须是一个INT（整数）类型的值。
  - type是时间单位，可以是YEAR（年）、QUARTER（季度）、MONTH（月）、WEEK（周）、DAY（日）。如果将DATE类型的值设置为HOUR（小时）、MINUTE（分钟）或SECOND（秒），则会返回错误。
- boundary：可选项。用于指定返回时间间隔的起始（FLOOR）还是结束（CEIL）。有效值：FLOOR（下限）、CEIL（上限）。如果未指定此参数，默认值为FLOOR。

## 返回值

返回DATE类型的值。

## 使用说明

时间间隔起始于公元0001年01月01日00:00:00。

## 示例

示例1：将给定时间转换为5年时间间隔的起始点，不指定boundary参数。

```Plaintext
select date_slice('2022-04-26', interval 5 year);
+--------------------------------------------------+
| date_slice('2022-04-26', INTERVAL 5 year, floor) |
+--------------------------------------------------+
| 2021-01-01                                       |
+--------------------------------------------------+
```

示例2：将给定时间转换为5天时间间隔的结束点。

```Plaintext
select date_slice('0001-01-07', interval 5 day, CEIL);
+------------------------------------------------+
| date_slice('0001-01-07', INTERVAL 5 day, ceil) |
+------------------------------------------------+
| 0001-01-11                                     |
+------------------------------------------------+
```

以下示例基于test_all_type_select表提供。

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

示例3：尝试将给定的DATE值转换为5秒时间间隔的起始点。

```Plaintext
select date_slice(id_date, interval 5 second, FLOOR)
from test_all_type_select
order by id_int;
ERROR 1064 (HY000): can't use date_slice for date with time(hour/minute/second)
```

由于系统无法处理DATE值的秒部分，因此会返回错误。

示例4：将给定的DATE值转换为5天时间间隔的起始点。

```Plaintext
select date_slice(id_date, interval 5 day, FLOOR)
from test_all_type_select
order by id_int;
+--------------------------------------------+
| date_slice(id_date, INTERVAL 5 day, floor) |
+--------------------------------------------+
| 2052-12-24                                 |
| 2168-08-03                                 |
| 1737-02-04                                 |
| 2245-09-29                                 |
| 1889-10-25                                 |
+--------------------------------------------+
5 rows in set (0.14 sec)
```

示例5：将给定的DATE值转换为5天时间间隔的结束点。

```Plaintext
select date_slice(id_date, interval 5 day, CEIL)
from test_all_type_select
order by id_int;
+-------------------------------------------+
| date_slice(id_date, INTERVAL 5 day, ceil) |
+-------------------------------------------+
| 2052-12-29                                |
| 2168-08-08                                |
| 1737-02-09                                |
| 2245-10-04                                |
| 1889-10-30                                |
+-------------------------------------------+
5 rows in set (0.17 sec)
```
