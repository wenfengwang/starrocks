---
displayed_sidebar: English
---

# str2date

## 描述

根据指定的格式将字符串转换为 DATE 值。如果转换失败，则返回 NULL。

格式必须与 [date_format](./date_format.md) 中描述的格式一致。

此函数等效于 [str_to_date](../date-time-functions/str_to_date.md)，但返回类型不同。

## 语法

```Haskell
DATE str2date(VARCHAR str, VARCHAR format);
```

## 参数

`str`：要转换的时间表达式。它必须是 VARCHAR 类型。

`format`：用于返回值的格式。有关支持的格式，请参阅 [date_format](./date_format.md)。

## 返回值

返回 DATE 类型的值。

如果 `str` 或 `format` 为 NULL，则返回 NULL。

## 例子

```Plain
select str2date('2010-11-30 23:59:59', '%Y-%m-%d %H:%i:%s');
+------------------------------------------------------+
| str2date('2010-11-30 23:59:59', '%Y-%m-%d %H:%i:%s') |
+------------------------------------------------------+
| 2010-11-30                                           |
+------------------------------------------------------+
1 行受影响 (0.01 秒)