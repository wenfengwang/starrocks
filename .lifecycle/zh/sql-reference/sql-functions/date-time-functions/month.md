---
displayed_sidebar: English
---

# 月份

## 描述

返回指定日期的月份。返回值的范围是1至12。

日期参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT MONTH(DATETIME date)
```

## 示例

```Plain
MySQL > select month('1987-01-01');
+-----------------------------+
|month('1987-01-01 00:00:00') |
+-----------------------------+
|                           1 |
+-----------------------------+
```

## 关键字

MONTH
