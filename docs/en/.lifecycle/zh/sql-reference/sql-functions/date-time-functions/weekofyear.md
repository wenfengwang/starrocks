---
displayed_sidebar: English
---

# weekofyear

## 描述

返回给定日期在一年中的周数。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT WEEKOFYEAR(DATETIME date)
```

## 示例

```Plain
MySQL > select weekofyear('2008-02-20 00:00:00');
+-----------------------------------+
| weekofyear('2008-02-20 00:00:00') |
+-----------------------------------+
|                                 8 |
+-----------------------------------+
```

## 关键字

WEEKOFYEAR