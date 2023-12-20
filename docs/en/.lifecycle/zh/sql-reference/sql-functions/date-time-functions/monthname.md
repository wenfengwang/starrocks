---
displayed_sidebar: English
---

# MONTHNAME

## 描述

返回给定日期的月份名称。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
VARCHAR MONTHNAME(date)
```

## 示例

```Plain
MySQL > select MONTHNAME('2008-02-03 00:00:00');
+----------------------------------+
| MONTHNAME('2008-02-03 00:00:00') |
+----------------------------------+
| February                         |
+----------------------------------+
```

## 关键字

MONTHNAME, monthname