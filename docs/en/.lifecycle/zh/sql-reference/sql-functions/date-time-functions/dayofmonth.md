---
displayed_sidebar: English
---

# DAYOFMONTH

## 描述

获取日期中的日部分，返回一个范围在1到31之间的值。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT DAYOFMONTH(DATETIME date)
```

## 示例

```Plain
MySQL > select DAYOFMONTH('1987-01-31');
+-----------------------------------+
| DAYOFMONTH('1987-01-31 00:00:00') |
+-----------------------------------+
|                                31 |
+-----------------------------------+
```

## 关键字

DAYOFMONTH