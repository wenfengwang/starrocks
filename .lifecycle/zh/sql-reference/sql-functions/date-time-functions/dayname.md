---
displayed_sidebar: English
---

# 星期名

## 描述

返回对应于某个日期的星期名称。

日期参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
VARCHAR DAYNAME(date)
```

## 示例

```Plain
MySQL > select dayname('2007-02-03 00:00:00');
+--------------------------------+
| dayname('2007-02-03 00:00:00') |
+--------------------------------+
| Saturday                       |
+--------------------------------+
```

## 关键字

DAYNAME
