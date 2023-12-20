---
displayed_sidebar: English
---

# time_slice

## 描述

根据指定的时间粒度将给定时间转换为时间间隔的开始或结束。

该函数从 v2.3 版本开始支持。v2.5 版本支持将给定时间转换为时间间隔的结束。

## 语法

```Haskell
DATETIME time_slice(DATETIME dt, INTERVAL N type[, boundary])
```

## 参数

- `dt`：要转换的时间，DATETIME 类型。
- `INTERVAL N type`：时间粒度，例如 `interval 5 second`。
  - `N` 是时间间隔的长度。它必须是一个 INT 类型的值。
  - `type` 是单位，可以是 YEAR、QUARTER、MONTH、WEEK、DAY、HOUR、MINUTE、SECOND。
- `boundary`：可选。用于指定返回时间间隔的开始（`FLOOR`）还是结束（`CEIL`）。有效值：FLOOR、CEIL。如果未指定此参数，则默认为 `FLOOR`。该参数从 v2.5 版本开始支持。

## 返回值

返回 DATETIME 类型的值。

## 使用说明

时间间隔从公元 `0001-01-01 00:00:00` 开始。

## 示例

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
5 行返回 (0.06 秒)
```

示例 1：将给定的 DATETIME 值转换为 5 秒时间间隔的开始，不指定 `boundary` 参数。

```Plaintext
select time_slice(id_datetime, interval 5 second)
from test_all_type_select
order by id_int;

+---------------------------------------------------+
| time_slice(id_datetime, INTERVAL 5 second, FLOOR) |
+---------------------------------------------------+
| 1691-12-23 04:01:05                               |
| 2169-12-18 15:44:30                               |
| 1840-11-23 13:09:50                               |
| 1751-03-21 00:19:00                               |
| 1861-09-12 13:28:15                               |
+---------------------------------------------------+
5 行返回 (0.16 秒)
```

示例 2：将给定的 DATETIME 值转换为 5 天时间间隔的开始，`boundary` 设置为 FLOOR。

```Plaintext
select time_slice(id_datetime, interval 5 day, FLOOR)
from test_all_type_select
order by id_int;

+------------------------------------------------+
| time_slice(id_datetime, INTERVAL 5 day, FLOOR) |
+------------------------------------------------+
| 1691-12-22 00:00:00                            |
| 2169-12-16 00:00:00                            |
| 1840-11-21 00:00:00                            |
| 1751-03-18 00:00:00                            |
| 1861-09-12 00:00:00                            |
+------------------------------------------------+
5 行返回 (0.15 秒)
```

示例 3：将给定的 DATETIME 值转换为 5 天时间间隔的结束。

```Plaintext
select time_slice(id_datetime, interval 5 day, CEIL)
from test_all_type_select
order by id_int;

+-----------------------------------------------+
| time_slice(id_datetime, INTERVAL 5 day, CEIL) |
+-----------------------------------------------+
| 1691-12-27 00:00:00                           |
| 2169-12-21 00:00:00                           |
| 1840-11-26 00:00:00                           |
| 1751-03-23 00:00:00                           |
| 1861-09-17 00:00:00                           |
+-----------------------------------------------+
5 行返回 (0.12 秒)
```