---
displayed_sidebar: "Chinese"
---

# dayofweek

## 描述

返回给定日期的工作日索引。例如，星期日的索引为1，星期一的索引为2，星期六的索引为7。

`date` 参数必须是 DATE 或 DATETIME 类型，或者是可转换为 DATE 或 DATETIME 值的有效表达式。

## 语法

```Haskell
INT dayofweek(DATETIME date)
```

## 示例

```Plain Text
MySQL > select dayofweek('2019-06-25');
+----------------------------------+
| dayofweek('2019-06-25 00:00:00') |
+----------------------------------+
|                                3 |
+----------------------------------+

MySQL > select dayofweek(cast(20190625 as date));
+-----------------------------------+
| dayofweek(CAST(20190625 AS DATE)) |
+-----------------------------------+
|                                 3 |
+-----------------------------------+
```

## 关键字

DAYOFWEEK