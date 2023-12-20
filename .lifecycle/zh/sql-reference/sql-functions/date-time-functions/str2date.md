---
displayed_sidebar: English
---

# 字符串转日期

## 描述

根据指定的格式将字符串转换成 DATE 类型的值。如果转换失败，将返回 NULL。

所用的格式必须与[date_format](./date_format.md)中描述的一致。

该函数与 [str_to_date](../date-time-functions/str_to_date.md) 功能相同，但返回类型不同。

## 语法

```Haskell
DATE str2date(VARCHAR str, VARCHAR format);
```

## 参数

str：想要转换的时间表达式。必须是 VARCHAR 类型。

`format`：用来返回值的格式。支持的格式详见 [date_format](./date_format.md)。

## 返回值

返回 DATE 类型的值。

如果 str 或 format 为 NULL，将返回 NULL。

## 示例

```Plain
select str2date('2010-11-30 23:59:59', '%Y-%m-%d %H:%i:%s');
+------------------------------------------------------+
| str2date('2010-11-30 23:59:59', '%Y-%m-%d %H:%i:%s') |
+------------------------------------------------------+
| 2010-11-30                                           |
+------------------------------------------------------+
1 row in set (0.01 sec)
```
