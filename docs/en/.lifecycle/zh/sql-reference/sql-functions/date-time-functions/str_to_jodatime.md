---
displayed_sidebar: English
---

# str_to_jodatime

## 描述

将符合 Joda 格式的字符串转换为指定 Joda DateTime 格式的 DATETIME 值，例如 `yyyy-MM-dd HH:mm:ss`。

## 语法

```Haskell
DATETIME str_to_jodatime(VARCHAR str, VARCHAR format)
```

## 参数

- `str`：您想要转换的时间表达式。它必须是 VARCHAR 类型。
- `format`：返回的 DATETIME 值的 Joda DateTime 格式。有关可用格式的信息，请参见 [Joda DateTime](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)。

## 返回值

- 如果解析输入字符串成功，返回一个 DATETIME 值。
- 如果解析输入字符串失败，返回 `NULL`。

## 示例

示例 1：将字符串 `2014-12-21 12:34:56` 转换为 `yyyy-MM-dd HH:mm:ss` 格式的 DATETIME 值。

```SQL
MySQL > select str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss');
+--------------------------------------------------------------+
| str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss') |
+--------------------------------------------------------------+
| 2014-12-21 12:34:56                                          |
+--------------------------------------------------------------+
```

示例 2：将带有文本风格月份的字符串 `21/December/23 12:34:56` 转换为 `dd/MMMM/yy HH:mm:ss` 格式的 DATETIME 值。

```SQL
MySQL > select str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss');
+------------------------------------------------------------------+
| str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss') |
+------------------------------------------------------------------+
| 2023-12-21 12:34:56                                              |
+------------------------------------------------------------------+
```

示例 3：将精确到毫秒的字符串 `21/December/23 12:34:56.123` 转换为 `dd/MMMM/yy HH:mm:ss.SSS` 格式的 DATETIME 值。

```SQL
MySQL > select str_to_jodatime('21/December/23 12:34:56.123', 'dd/MMMM/yy HH:mm:ss.SSS');
+--------------------------------------------------------------------------+
| str_to_jodatime('21/December/23 12:34:56.123', 'dd/MMMM/yy HH:mm:ss.SSS') |
+--------------------------------------------------------------------------+
| 2023-12-21 12:34:56.123000                                               |
+--------------------------------------------------------------------------+
```

## 关键字

STR_TO_JODATIME, DATETIME