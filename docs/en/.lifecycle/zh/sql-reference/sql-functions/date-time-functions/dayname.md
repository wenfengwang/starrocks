---
displayed_sidebar: English
---

# DAYNAME

## 描述

返回对应于日期的星期几。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
VARCHAR DAYNAME(date)
```

## 示例

```Plain
MySQL > select DAYNAME('2007-02-03 00:00:00');
+--------------------------------+
| DAYNAME('2007-02-03 00:00:00') |
+--------------------------------+
| Saturday                       |
+--------------------------------+
```

## 关键字

DAYNAME