---
displayed_sidebar: English
---

# DAYNAME

## 描述

返回与日期对应的星期几。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
VARCHAR DAYNAME(date)
```

## 例子

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