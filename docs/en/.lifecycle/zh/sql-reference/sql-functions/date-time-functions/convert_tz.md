---
displayed_sidebar: English
---

# convert_tz

## 描述

将 DATE 或 DATETIME 值从一个时区转换为另一个时区。

此函数可能会针对不同的时区返回不同的结果。有关详细信息，请参阅 [配置时区](../../../administration/timezone.md)。

## 语法

```Haskell
DATETIME CONVERT_TZ(DATETIME|DATE dt, VARCHAR from_tz, VARCHAR to_tz)
```

## 参数

- `dt`：要转换的 DATE 或 DATETIME 值。

- `from_tz`：源时区。支持 VARCHAR。时区可以用两种格式表示：一种是时区数据库（例如，Asia/Shanghai），另一种是 UTC 偏移量（例如，+08:00）。

- `to_tz`：目标时区。支持 VARCHAR。其格式与 `from_tz` 相同。

## 返回值

返回 DATETIME 数据类型的值。如果输入是 DATE 值，则会转换为 DATETIME 值。如果任何输入参数无效或为 NULL，则此函数将返回 NULL。

## 使用说明

有关时区数据库，请参阅 [tz 数据库时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) （在 Wikipedia 中）。

## 例子

示例 1：将上海的日期时间转换为 Los_Angeles。

```plaintext
select convert_tz('2019-08-01 13:21:03', 'Asia/Shanghai', 'America/Los_Angeles');
+---------------------------------------------------------------------------+
| convert_tz('2019-08-01 13:21:03', 'Asia/Shanghai', 'America/Los_Angeles') |
+---------------------------------------------------------------------------+
| 2019-07-31 22:21:03                                                       |
+---------------------------------------------------------------------------+
1 行受影响 (0.00 秒)                                                       |
```

示例 2：将上海的日期转换为 Los_Angeles。

```plaintext
select convert_tz('2019-08-01', 'Asia/Shanghai', 'America/Los_Angeles');
+------------------------------------------------------------------+
| convert_tz('2019-08-01', 'Asia/Shanghai', 'America/Los_Angeles') |
+------------------------------------------------------------------+
| 2019-07-31 09:00:00                                              |
+------------------------------------------------------------------+
1 行受影响 (0.00 秒)
```

示例 3：将 UTC+08:00 中的日期时间转换为 Los_Angeles。

```plaintext
select convert_tz('2019-08-01 13:21:03', '+08:00', 'America/Los_Angeles');
+--------------------------------------------------------------------+
| convert_tz('2019-08-01 13:21:03', '+08:00', 'America/Los_Angeles') |
+--------------------------------------------------------------------+
| 2019-07-31 22:21:03                                                |
+--------------------------------------------------------------------+
1 行受影响 (0.00 秒)
```

## 关键字

CONVERT_TZ、时区、时区