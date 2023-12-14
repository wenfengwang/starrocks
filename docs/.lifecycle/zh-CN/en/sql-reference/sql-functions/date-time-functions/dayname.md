---
displayed_sidebar: "Chinese"
---

# dayname

## 描述

返回与日期对应的一天。

`date`参数必须是DATE或DATETIME类型。

## 语法

```Haskell
VARCHAR DAYNAME(date)
```

## 示例

```Plain Text
MySQL > select dayname('2007-02-03 00:00:00');
+--------------------------------+
| dayname('2007-02-03 00:00:00') |
+--------------------------------+
| Saturday                       |
+--------------------------------+
```

## 关键词

DAYNAME