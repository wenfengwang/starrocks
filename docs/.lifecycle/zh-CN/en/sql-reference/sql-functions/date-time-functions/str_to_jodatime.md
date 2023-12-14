---
displayed_sidebar: "Chinese"
---

# str_to_jodatime

## 描述

将Joda格式的字符串转换为指定的Joda DateTime格式（例如`yyyy-MM-dd HH:mm:ss`）的DATETIME值。

## 语法

```Haskell
DATETIME str_to_jodatime(VARCHAR str, VARCHAR format)
```

## 参数

- `str`：要转换的时间表达式。必须是VARCHAR类型。
- `format`：要返回的DATETIME值的Joda DateTime格式。有关可用格式的信息，请参见[Joda DateTime](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)。

## 返回值

- 如果成功解析输入字符串，则返回一个DATETIME值。
- 如果解析输入字符串失败，则返回`NULL`。

## 示例

示例1：将字符串`2014-12-21 12:34:56`转换为`yyyy-MM-dd HH:mm:ss`格式的DATETIME值。

```SQL
MySQL > select str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss');
+--------------------------------------------------------------+
| str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss') |
+--------------------------------------------------------------+
| 2014-12-21 12:34:56                                          |
+--------------------------------------------------------------+
```

示例2：将带有文本样式月份的字符串`21/December/23 12:34:56`转换为`dd/MMMM/yy HH:mm:ss`格式的DATETIME值。

```SQL
MySQL > select str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss');
+------------------------------------------------------------------+
| str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss') |
+------------------------------------------------------------------+
| 2023-12-21 12:34:56                                              |
+------------------------------------------------------------------+
```

示例3：将精确到毫秒的字符串`21/December/23 12:34:56.123`转换为`dd/MMMM/yy HH:mm:ss.SSS`格式的DATETIME值。

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