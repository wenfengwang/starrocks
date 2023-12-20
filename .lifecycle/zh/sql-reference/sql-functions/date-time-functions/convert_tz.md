---
displayed_sidebar: English
---

# 转换时区（convert_tz）

## 描述

此功能将 DATE 或 DATETIME 值从一个时区转换到另一个时区。

该函数可能会针对不同的时区返回不同的结果。更多信息请参见[配置时区](../../../administration/timezone.md)指南。

## 语法

```Haskell
DATETIME CONVERT_TZ(DATETIME|DATE dt, VARCHAR from_tz, VARCHAR to_tz)
```

## 参数

- dt：需要转换的 DATE 或 DATETIME 值。

- from_tz：源时区。支持 VARCHAR 类型。时区可用两种格式表示：一种是时区数据库名称（例如，Asia/Shanghai），另一种是 UTC 偏移量（例如，+08:00）。

- to_tz：目标时区。支持 VARCHAR 类型。格式与 from_tz 相同。

## 返回值

返回 DATETIME 数据类型的值。如果输入是 DATE 类型，将被转换为 DATETIME 类型。若任一输入参数无效或为 NULL，则该函数返回 NULL。

## 使用说明

关于时区数据库，请参见 [tz 数据库时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)（见维基百科）。

## 示例

示例 1：将上海的 datetime 转换为洛杉矶时间。

```plaintext
select convert_tz('2019-08-01 13:21:03', 'Asia/Shanghai', 'America/Los_Angeles');
+---------------------------------------------------------------------------+
| convert_tz('2019-08-01 13:21:03', 'Asia/Shanghai', 'America/Los_Angeles') |
+---------------------------------------------------------------------------+
| 2019-07-31 22:21:03                                                       |
+---------------------------------------------------------------------------+
1 row in set (0.00 sec)                                                       |
```

示例 2：将上海的 date 转换为洛杉矶时间。

```plaintext
select convert_tz('2019-08-01', 'Asia/Shanghai', 'America/Los_Angeles');
+------------------------------------------------------------------+
| convert_tz('2019-08-01', 'Asia/Shanghai', 'America/Los_Angeles') |
+------------------------------------------------------------------+
| 2019-07-31 09:00:00                                              |
+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

示例 3：将 UTC+08:00 的 datetime 转换为洛杉矶时间。

```plaintext
select convert_tz('2019-08-01 13:21:03', '+08:00', 'America/Los_Angeles');
+--------------------------------------------------------------------+
| convert_tz('2019-08-01 13:21:03', '+08:00', 'America/Los_Angeles') |
+--------------------------------------------------------------------+
| 2019-07-31 22:21:03                                                |
+--------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## 关键字

CONVERT_TZ，时区转换，时区
