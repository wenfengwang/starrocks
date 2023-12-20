---
displayed_sidebar: English
---

# to_tera_timestamp

## 描述

将指定的 VARCHAR 值转换成特定格式的 DATETIME 值。

## 语法

```Haskell
DATETIME to_tera_timestamp(VARCHAR str, VARCHAR format)
```

## 参数

- str：想要转换的时间表达式。必须是 VARCHAR 类型。

- format：返回的 DATETIME 值的格式。

  以下表格描述了格式元素。

  |元素|描述|
|---|---|
  |[ \r \n \t - / , . ;]|标点符号被忽略。|
  |dd|一月中的哪一天。有效值：1 - 31。|
  |hh|一天中的某个时刻。有效值：1 - 12。|
  |hh24|一天中的某个时刻。有效值：0 - 23。|
  |mi|分钟。有效值：0 - 59。|
  |毫米|月。有效值：01 - 12。|
  |ss|第二。有效值：0 - 59。|
  |yyyy|4 位数年份。|
  |yy|2 位数年份。|
  |am|经络指示器。|
  |pm|经络指示器。|

## 示例

以下示例将 VARCHAR 值“1988/04/08 2:3:4”转换为“yyyy/mm/dd hh24:mi:ss”格式的 DATETIME 值。

```SQL
MySQL > select to_tera_timestamp("1988/04/08 2:3:4","yyyy/mm/dd hh24:mi:ss");
+-----------------------------------------------------------+
| to_tera_timestamp('1988/04/08 2:3:4', 'yyyy/mm/dd hh24:mi:ss') |
+-----------------------------------------------------------+
| 1988-04-08 02:03:04                                       |
+-----------------------------------------------------------+
```

## 关键字

TO_TERA_TIMESTAMP
