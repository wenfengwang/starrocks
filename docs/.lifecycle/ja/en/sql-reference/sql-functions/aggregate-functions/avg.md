---
displayed_sidebar: English
---

# avg

## 説明

選択したフィールドの平均値を返します。

オプションのフィールドDISTINCTパラメータを使用して、加重平均を返すことができます。

## 構文

```Haskell
AVG([DISTINCT] expr)
```

## 例

```plain text
MySQL > SELECT datetime, AVG(cost_time)
FROM log_statis
GROUP BY datetime;
+---------------------+--------------------+
| datetime            | AVG(`cost_time`)   |
+---------------------+--------------------+
| 2019-07-03 21:01:20 | 25.827794561933533 |
+---------------------+--------------------+

MySQL > SELECT datetime, AVG(DISTINCT cost_time)
FROM log_statis
GROUP BY datetime;
+---------------------+---------------------------+
| datetime            | AVG(DISTINCT `cost_time`) |
+---------------------+---------------------------+
| 2019-07-04 02:23:24 |        20.666666666666668 |
+---------------------+---------------------------+

```

## キーワード

AVG
