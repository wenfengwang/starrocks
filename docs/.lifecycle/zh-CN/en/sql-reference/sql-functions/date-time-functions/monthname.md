---
displayed_sidebar: "Chinese"
---

# monthname

## 描述

返回给定日期的月份名称。

`date`参数必须是DATE或DATETIME类型。

## 语法

```Haskell
VARCHAR MONTHNAME(date)
```

## 示例

```Plain Text
MySQL > select monthname('2008-02-03 00:00:00');
+----------------------------------+
| monthname('2008-02-03 00:00:00') |
+----------------------------------+
| February                         |
+----------------------------------+
```

## 关键词

MONTHNAME, monthname