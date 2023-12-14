---
displayed_sidebar: "Chinese"
---

# date_slice

## 描述

根据指定的时间粒度，将给定时间转换为时间间隔的开始或结束。

该函数从 v2.5 版本开始支持。

## 语法

```Haskell
DATE date_slice(DATE dt, INTERVAL N type[, boundary])
```

## 参数

- `dt`：要转换的时间，DATE 类型。
- `INTERVAL N type`：时间粒度，例如 `interval 5 day`。
  - `N` 为时间间隔的长度，必须是 INT 类型的值。
  - `type` 为单位，可以是 YEAR、QUARTER、MONTH、WEEK、DAY。 如果对于 DATE 值设置 `type` 为 HOUR、MINUTE 或 SECOND，则返回错误。
- `boundary`：可选。用于指定是返回时间间隔的开始（`FLOOR`）还是结束（`CEIL`）。有效值为：FLOOR、CEIL。如果未指定该参数，则默认为 `FLOOR`。

## 返回值

返回 DATE 类型的值。

## 使用注意事项

时间间隔始于公元 `0001-01-01 00:00:00`。

## 示例

示例 1：将给定时间转换为 5 年时间间隔的开始，而不指定 `boundary` 参数。

```Plaintext
select date_slice('2022-04-26', interval 5 year);
+--------------------------------------------------+
| date_slice('2022-04-26', INTERVAL 5 year, floor) |
+--------------------------------------------------+
| 2021-01-01                                       |
+--------------------------------------------------+
```

示例 2：将给定时间转换为 5 天时间间隔的结束。

```Plaintext
select date_slice('0001-01-07', interval 5 day, CEIL);
+------------------------------------------------+
| date_slice('0001-01-07', INTERVAL 5 day, ceil) |
+------------------------------------------------+
| 0001-01-11                                     |
+------------------------------------------------+
```

以下示例基于 `test_all_type_select` 表提供。

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

示例 3：将给定 DATE 值转换为 5 秒时间间隔的开始。

```Plaintext
select date_slice(id_date, interval 5 second, FLOOR)
from test_all_type_select
order by id_int;
ERROR 1064 (HY000): can't use date_slice for date with time(hour/minute/second)
```

返回错误，因为系统无法找到 DATE 值的秒部分。

示例 4：将给定 DATE 值转换为 5 天时间间隔的开始。

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

示例 5：将给定 DATE 值转换为 5 天时间间隔的结束。

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