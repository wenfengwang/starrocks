---
displayed_sidebar: English
---

# avg

## 描述

返回所选字段的平均值。

可选的 DISTINCT 参数可以用来返回加权平均值。

## 语法

```Haskell
AVG([DISTINCT] expr)
```

## 示例

```plain
MySQL > SELECT datetime, AVG(cost_time)
FROM log_statis
GROUP BY datetime;
+---------------------+--------------------+
| datetime            | avg(`cost_time`)   |
+---------------------+--------------------+
| 2019-07-03 21:01:20 | 25.827794561933533 |
+---------------------+--------------------+

MySQL > SELECT datetime, AVG(DISTINCT cost_time)
FROM log_statis
GROUP BY datetime;
+---------------------+---------------------------+
| datetime            | avg(DISTINCT `cost_time`) |
+---------------------+---------------------------+
| 2019-07-04 02:23:24 |        20.666666666666668 |
+---------------------+---------------------------+

```

## 关键字

AVG