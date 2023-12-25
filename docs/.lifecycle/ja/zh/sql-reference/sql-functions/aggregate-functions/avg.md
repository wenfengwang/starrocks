---
displayed_sidebar: Chinese
---

# avg

## 機能

選択したフィールドの平均値を返すために使用されます。

## 文法

```Haskell
AVG([DISTINCT] expr)
```

## パラメータ説明

`expr`: 選択された表現。DISTINCT オプションを使用することができます。

## 戻り値の説明

戻り値は数値型です。

## 例

```plain text
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
