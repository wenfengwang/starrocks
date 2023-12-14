---
displayed_sidebar: "Chinese"
---

# dayofmonth

## Description

获取日期的天部分，并返回一个值，范围从1到31。

`date` 参数必须是 DATE 或 DATETIME 类型。

## Syntax

```Haskell
INT DAYOFMONTH(DATETIME date)
```

## Examples

```Plain Text
MySQL > select dayofmonth('1987-01-31');
+-----------------------------------+
| dayofmonth('1987-01-31 00:00:00') |
+-----------------------------------+
|                                31 |
+-----------------------------------+
```

## keyword

DAYOFMONTH