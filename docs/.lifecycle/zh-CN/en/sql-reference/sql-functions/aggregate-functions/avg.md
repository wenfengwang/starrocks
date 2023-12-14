---
displayed_sidebar: "Chinese"
---

# 平均值

## 描述

返回所选字段的平均值。

可选的DISTINCT参数可用于返回加权平均值。

## 语法

```Haskell
AVG([DISTINCT] expr)
```

## 示例

```plain text
MySQL > SELECT datetime, AVG(cost_time)
FROM log_statis
group by datetime;
+---------------------+--------------------+
| datetime            | avg(`cost_time`)   |
+---------------------+--------------------+
| 2019-07-03 21:01:20 | 25.827794561933533 |
+---------------------+--------------------+

MySQL > SELECT datetime, AVG(distinct cost_time)
FROM log_statis
group by datetime;
+---------------------+---------------------------+
| datetime            | avg(DISTINCT `cost_time`) |
+---------------------+---------------------------+
| 2019-07-04 02:23:24 |        20.666666666666668 |
+---------------------+---------------------------+

```

## 关键词

AVG