---
displayed_sidebar: English
---

# 月份

## 描述

返回给定日期的月份。返回值范围为 1 到 12。

`date` 参数必须是 DATE 或 DATETIME 类型。

## 语法

```Haskell
INT MONTH(DATETIME date)
```

## 例子

```Plain Text
MySQL > select month('1987-01-01');
+-----------------------------+
|month('1987-01-01 00:00:00') |
+-----------------------------+
|                           1 |
+-----------------------------+
```

## 关键词

月份
