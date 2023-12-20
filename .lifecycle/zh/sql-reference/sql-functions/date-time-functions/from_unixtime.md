---
displayed_sidebar: English
---

# from_unixtime

## 描述

将UNIX时间戳转换成指定的时间格式。默认格式是 `yyyy-MM-dd HH:mm:ss`。同时也支持[date_format](./date_format.md)中定义的格式。

目前，string_format支持以下格式：

```plain
%Y: Year  e.g.: 2014, 1900
%m: Month   e.g.: 12, 09
%d: Day  e.g.: 11, 01
%H: Hour  e.g.: 23, 01, 12
%i: Minute  e.g.: 05, 11
%s: Second  e.g.: 59, 01
```

不支持的其他格式会返回NULL。

## 语法

```Haskell
VARCHAR from_unixtime(BIGINT unix_timestamp[, VARCHAR string_format])
```

## 参数

- unix_timestamp：需要转换的UNIX时间戳。它必须是BIGINT类型。如果指定的时间戳小于0或大于253402243199，将返回NULL。即时间戳的有效范围是1970-01-01 00:00:00至9999-12-30 11:59:59（可能因时区有所变化）。

- string_format：期望的时间格式。

## 返回值

返回VARCHAR类型的DATETIME或DATE值。如果string_format指定为DATE格式，那么返回的是VARCHAR类型的DATE值。

如果时间戳超出了有效范围或者string_format是无效的，将返回NULL。

## 示例

```plain
MySQL > select from_unixtime(1196440219);
+---------------------------+
| from_unixtime(1196440219) |
+---------------------------+
| 2007-12-01 00:30:19       |
+---------------------------+

MySQL > select from_unixtime(1196440219, 'yyyy-MM-dd HH:mm:ss');
+--------------------------------------------------+
| from_unixtime(1196440219, 'yyyy-MM-dd HH:mm:ss') |
+--------------------------------------------------+
| 2007-12-01 00:30:19                              |
+--------------------------------------------------+

MySQL > select from_unixtime(1196440219, '%Y-%m-%d');
+-----------------------------------------+
| from_unixtime(1196440219, '%Y-%m-%d')   |
+-----------------------------------------+
| 2007-12-01                              |
+-----------------------------------------+

MySQL > select from_unixtime(1196440219, '%Y-%m-%d %H:%i:%s');
+--------------------------------------------------+
| from_unixtime(1196440219, '%Y-%m-%d %H:%i:%s')   |
+--------------------------------------------------+
| 2007-12-01 00:30:19                              |
+--------------------------------------------------+
```

## 关键字

FROM_UNIXTIME，FROM，UNIXTIME
