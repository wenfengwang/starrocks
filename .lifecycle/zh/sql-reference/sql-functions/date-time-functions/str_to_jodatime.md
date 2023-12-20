---
displayed_sidebar: English
---

# str_to_jodatime

## 描述

将遵循Joda格式的字符串转换成指定的Joda DateTime格式的DATETIME值，如 yyyy-MM-dd HH:mm:ss。

## 语法

```Haskell
DATETIME str_to_jodatime(VARCHAR str, VARCHAR format)
```

## 参数

- str：您想要转换的时间表达式。它必须是VARCHAR类型。
- `format`：要返回的DATETIME值的Joda DateTime格式。关于可用格式的详细信息，请参见[Joda DateTime](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)文档。

## 返回值

- 如果输入字符串的解析成功，将返回DATETIME值。
- 如果输入字符串的解析失败，将返回NULL。

## 示例

示例1：将字符串"2014-12-21 12:34:56"转换为yyyy-MM-dd HH:mm:ss格式的DATETIME值。

```SQL
MySQL > select str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss');
+--------------------------------------------------------------+
| str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss') |
+--------------------------------------------------------------+
| 2014-12-21 12:34:56                                          |
+--------------------------------------------------------------+
```

示例2：将包含文本月份的字符串"21/December/23 12:34:56"转换为dd/MMMM/yy HH:mm:ss格式的DATETIME值。

```SQL
MySQL > select str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss');
+------------------------------------------------------------------+
| str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss') |
+------------------------------------------------------------------+
| 2023-12-21 12:34:56                                              |
+------------------------------------------------------------------+
```

示例3：将精确到毫秒的字符串"21/December/23 12:34:56.123"转换为dd/MMMM/yy HH:mm:ss.SSS格式的DATETIME值。

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
