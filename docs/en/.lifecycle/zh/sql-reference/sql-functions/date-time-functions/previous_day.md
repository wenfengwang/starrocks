---
displayed_sidebar: English
---

# previous_day

## 描述

返回输入日期（DATE 或 DATETIME）之前最近的指定星期几（DOW）的日期。例如，`previous_day('2023-04-06', 'Monday')` 返回“2023-04-06”之前最近的一个星期一的日期。

该函数从 v3.1 版本开始支持。它与 [next_day](./next_day.md) 函数相反。

## 语法

```SQL
DATE previous_day(DATETIME|DATE date_expr, VARCHAR dow)
```

## 参数

- `date_expr`：输入日期。它必须是有效的 DATE 或 DATETIME 表达式。
- `dow`：星期几。有效值包括多种区分大小写的缩写：

  |DOW_FULL|DOW_2|DOW_3|
|---|---|---|
  |Sunday|Su|Sun|
  |Monday|Mo|Mon|
  |Tuesday|Tu|Tue|
  |Wednesday|We|Wed|
  |Thursday|Th|Thu|
  |Friday|Fr|Fri|
  |Saturday|Sa|Sat|

## 返回值

返回 DATE 类型的值。

任何无效的 `dow` 都会导致错误。`dow` 区分大小写。

如果传入无效日期或 NULL 参数，则返回 NULL。

## 示例

```Plain
-- 返回 2023-04-06 之前最近的一个星期一的日期。2023-04-06 是星期四，最近的一个星期一是 2023-04-03。

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

PREVIOUS_DAY, PREVIOUS, previousday