---
displayed_sidebar: English
---

# 前一天

## 描述

返回输入日期（DATE 或 DATETIME）之前最近的一个指定星期几（DOW）的日期。例如，previous_day('2023-04-06', 'Monday') 会返回“2023-04-06”之前最近一个星期一的日期。

该函数从v3.1版本开始支持。它的作用与 [next_day](./next_day.md) 相反。

## 语法

```SQL
DATE previous_day(DATETIME|DATE date_expr, VARCHAR dow)
```

## 参数

- date_expr：输入的日期。它必须是一个有效的 DATE 或 DATETIME 表达式。
- dow：星期几。有效值包括多个区分大小写的缩写：

  |DOW_FULL|DOW_2|DOW_3|
|---|---|---|
  |周日|苏|周日|
  |星期一|Mo|星期一|
  |星期二|星期二|星期二|
  |星期三|我们|星期三|
  |星期四|星期四|星期四|
  |星期五|Fr|星期五|
  |星期六|星期六|星期六|

## 返回值

返回一个 DATE 类型的值。

任何无效的 dow 将会导致错误。dow 是区分大小写的。

如果传入了无效的日期或 NULL 参数，则返回 NULL。

## 示例

```Plain
-- Return the date of the previous Monday that occurred before 2023-04-06. 2023-04-06 is Thursday and the date of the previous Monday is 2023-04-03.

MySQL > select previous_day('2023-04-06', 'Monday');
+--------------------------------------+
| previous_day('2023-04-06', 'Monday') |
+--------------------------------------+
| 2023-04-03                           |
+--------------------------------------+

MySQL > select previous_day('2023-04-06', 'Tue');
+-----------------------------------+
| previous_day('2023-04-06', 'Tue') |
+-----------------------------------+
| 2023-04-04                        |
+-----------------------------------+

MySQL > select previous_day('2023-04-06 20:13:14', 'Fr');
+-------------------------------------------+
| previous_day('2023-04-06 20:13:14', 'Fr') |
+-------------------------------------------+
| 2023-03-31                                |
+-------------------------------------------+
```

## 关键词

PREVIOUS_DAY, 前一天, previous_day
