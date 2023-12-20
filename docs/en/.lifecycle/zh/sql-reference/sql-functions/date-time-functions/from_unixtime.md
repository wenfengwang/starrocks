---
displayed_sidebar: English
---

# from_unixtime

## 描述

将 UNIX 时间戳转换为所需的时间格式。默认格式为 `yyyy-MM-dd HH:mm:ss`。它还支持 [date_format](./date_format.md) 中的格式。

目前，`string_format` 支持以下格式：

```plain
%Y: Year  例如：2014, 1900
%m: Month   例如：12, 09
%d: Day  例如：11, 01
%H: Hour  例如：23, 01, 12
%i: Minute  例如：05, 11
%s: Second  例如：59, 01
```

其他格式无效，将返回 NULL。

## 语法

```Haskell
VARCHAR from_unixtime(BIGINT unix_timestamp[, VARCHAR string_format])
```

## 参数

- `unix_timestamp`：要转换的 UNIX 时间戳。它必须是 BIGINT 类型。如果指定的时间戳小于 0 或大于 253402243199，则返回 NULL。也就是说，时间戳的有效范围是 `1970-01-01 00:00:00` 到 `9999-12-30 11:59:59`（因时区而异）。

- `string_format`：所需的时间格式。

## 返回值

返回 VARCHAR 类型的 DATETIME 或 DATE 值。如果 `string_format` 指定 DATE 格式，则返回 VARCHAR 类型的 DATE 值。

如果时间戳超出有效范围或者 `string_format` 无效，则返回 NULL。

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

## 关键词

FROM_UNIXTIME, FROM, UNIXTIME