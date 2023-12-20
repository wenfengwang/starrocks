---
displayed_sidebar: English
---

# to_tera_timestamp

## 描述

将指定的 VARCHAR 值转换成指定格式的 DATETIME 值。

## 语法

```Haskell
DATETIME to_tera_timestamp(VARCHAR str, VARCHAR format)
```

## 参数

- `str`：要转换的时间表达式。它必须是 VARCHAR 类型。

- `format`：要返回的 DATETIME 值的格式。

  下表描述了格式元素。

  |**元素**|**描述**|
|---|---|
  |[ \r \n \t - / , . ;]|标点符号被忽略。|
  |dd|月份中的日。有效值：`1` - `31`。|
  |hh|小时。有效值：`1` - `12`。|
  |hh24|24小时制的小时。有效值：`0` - `23`。|
  |mi|分钟。有效值：`0` - `59`。|
  |mm|月份。有效值：`01` - `12`。|
  |ss|秒。有效值：`0` - `59`。|
  |yyyy|4位数年份。|
  |yy|2位数年份。|
  |am|上午/下午标识符。|
  |pm|上午/下午标识符。|

## 示例

以下示例将 VARCHAR 值 `1988/04/08 2:3:4` 转换为 `yyyy/mm/dd hh24:mi:ss` 格式的 DATETIME 值。

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