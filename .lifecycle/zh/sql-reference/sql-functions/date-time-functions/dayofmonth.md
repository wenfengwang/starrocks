---
displayed_sidebar: English
---

# 获取月份中的日

## 描述

提取日期中的日部分，并返回一个介于1至31之间的值。

日期参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT DAYOFMONTH(DATETIME date)
```

## 示例

```Plain
MySQL > select dayofmonth('1987-01-31');
+-----------------------------------+
| dayofmonth('1987-01-31 00:00:00') |
+-----------------------------------+
|                                31 |
+-----------------------------------+
```

## 关键字

DAYOFMONTH
