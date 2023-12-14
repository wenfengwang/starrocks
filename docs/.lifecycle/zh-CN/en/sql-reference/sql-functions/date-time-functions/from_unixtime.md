---
displayed_sidebar: "Chinese"
---

# from_unixtime

## Description

将UNIX时间戳转换为所需的时间格式。默认格式为`yyyy-MM-dd HH:mm:ss`。还支持[date_format](./date_format.md)中的格式。

目前，`string_format`支持以下格式：

```plain text
%Y: 年  例如：2014, 1900
%m: 月  例如：12, 09
%d: 日  例如：11, 01
%H: 时  例如：23, 01, 12
:i: 分 例如：05, 11
%s: 秒 例如：59, 01
```

其他格式无效，会返回NULL。

## 语法

```Haskell
VARCHAR from_unixtime(BIGINT unix_timestamp[, VARCHAR string_format])
```

## 参数

- `unix_timestamp`：要转换的UNIX时间戳。必须是BIGINT类型。如果指定的时间戳小于0或大于253402243199，则返回NULL。也就是说，时间戳的范围是`1970-01-01 00:00:00`至`9999-12-30 11:59:59`(由于时区不同而变化)。

- `string_format`：所需的时间格式。

## 返回值

返回VARCHAR类型的DATETIME或DATE值。如果`string_format`指定了DATE格式，则返回VARCHAR类型的DATE值。

如果时间戳超出值范围或`string_format`无效，则返回NULL。

## 示例

```plain text
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

FROM_UNIXTIME,FROM,UNIXTIME