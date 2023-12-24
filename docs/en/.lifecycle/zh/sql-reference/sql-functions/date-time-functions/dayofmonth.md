---
displayed_sidebar: English
---

# DAYOFMONTH

## 描述

获取日期中的日，并返回一个介于 1 到 31 之间的值。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT DAYOFMONTH(DATETIME date)
```

## 例子

```Plain Text
MySQL > select dayofmonth('1987-01-31');
+-----------------------------------+
| dayofmonth('1987-01-31 00:00:00') |
+-----------------------------------+
|                                31 |
+-----------------------------------+
```

## 关键词

DAYOFMONTH
