---
displayed_sidebar: English
---

# 一年中的第几天

## 描述

返回指定日期为一年中的第几天。

日期参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT DAYOFYEAR(DATETIME|DATE date)
```

## 示例

```Plain
MySQL > select dayofyear('2007-02-03 00:00:00');
+----------------------------------+
| dayofyear('2007-02-03 00:00:00') |
+----------------------------------+
|                               34 |
+----------------------------------+
```

## 关键字

DAYOFYEAR
