---
displayed_sidebar: English
---

# 生成序列

## 描述

在由开始值和结束值指定的区间内生成一系列数值，可选地指定步长。

generate_series() 是一个表函数。表函数能够为每个输入行返回一组行集。行集可以包含零行、一行或多行。每一行可以包含一列或多列。

要在 StarRocks 中使用 generate_series()，如果输入参数是常量，则必须将其包含在 TABLE 关键字中。如果输入参数是表达式，比如列名，则无需使用 TABLE 关键字（参见示例 5）。

该函数从 v3.1 版本开始支持。

## 语法

```SQL
generate_series(start, end [,step])
```

## 参数

- start：序列的起始值，必填。支持的数据类型为 INT、BIGINT 和 LARGEINT。
- end：序列的结束值，必填。支持的数据类型为 INT、BIGINT 和 LARGEINT。
- step：递增或递减的值，可选。支持的数据类型为 INT、BIGINT 和 LARGEINT。若未指定，默认步长为 1。步长可以是正数或负数，但不能为零。

这三个参数必须是相同的数据类型，例如 generate_series(INT start, INT end [, INT step])。从 v3.3 版本开始支持命名参数，所有参数均需按名称输入，如 name=>expr，例如 generate_series(start=>3, end=>7, step=>2)。

## 返回值

返回一系列数值，这些数值与输入参数 start 和 end 的类型一致。

- 当 step 为正时，如果 start 大于 end，则不返回任何行。相反，当 step 为负时，如果 start 小于 end，则不返回任何行。
- 如果 step 为 0，则返回错误。
- 该函数对空值的处理如下：如果任何输入参数是字面量 null，则报告错误。如果任何输入参数是表达式且表达式结果为 null，则不返回任何行（参见示例 5）。

## 示例

示例 1：生成 [2,5] 范围内的数值序列，以默认步长 1 升序排列。

```SQL
MySQL > select * from TABLE(generate_series(2, 5));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               3 |
|               4 |
|               5 |
+-----------------+
```

示例 2：以指定步长 2 生成 [2,5] 范围内的数值序列，升序排列。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, 2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

示例 3：以指定步长 -1 生成 [5,2] 范围内的数值序列，降序排列。

```SQL
MySQL > select * from TABLE(generate_series(5, 2, -1));
+-----------------+
| generate_series |
+-----------------+
|               5 |
|               4 |
|               3 |
|               2 |
+-----------------+
```

示例 4：当 step 为负且 start 小于 end 时，不返回任何行。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, -1));
Empty set (0.01 sec)
```

示例 5：使用表列作为 generate_series() 的输入参数。在这种情况下，不需要将 TABLE() 与 generate_series() 一起使用。

```SQL
CREATE TABLE t_numbers(start INT, end INT)
DUPLICATE KEY (start)
DISTRIBUTED BY HASH(start) BUCKETS 1;

INSERT INTO t_numbers VALUES
(1, 3),
(5, 2),
(NULL, 10),
(4, 7),
(9,6);

SELECT * FROM t_numbers;
+-------+------+
| start | end  |
+-------+------+
|  NULL |   10 |
|     1 |    3 |
|     4 |    7 |
|     5 |    2 |
|     9 |    6 |
+-------+------+

-- Generate multiple rows for rows (1,3) and (4,7) with step 1.
SELECT * FROM t_numbers, generate_series(t_numbers.start, t_numbers.end);
+-------+------+-----------------+
| start | end  | generate_series |
+-------+------+-----------------+
|     1 |    3 |               1 |
|     1 |    3 |               2 |
|     1 |    3 |               3 |
|     4 |    7 |               4 |
|     4 |    7 |               5 |
|     4 |    7 |               6 |
|     4 |    7 |               7 |
+-------+------+-----------------+

-- Generate multiple rows for rows (5,2) and (9,6) with step -1.
SELECT * FROM t_numbers, generate_series(t_numbers.start, t_numbers.end, -1);
+-------+------+-----------------+
| start | end  | generate_series |
+-------+------+-----------------+
|     5 |    2 |               5 |
|     5 |    2 |               4 |
|     5 |    2 |               3 |
|     5 |    2 |               2 |
|     9 |    6 |               9 |
|     9 |    6 |               8 |
|     9 |    6 |               7 |
|     9 |    6 |               6 |
+-------+------+-----------------+
```
示例 6：使用命名参数，以指定步长 2 生成 [2,5] 范围内的数值序列，升序排列。

```SQL
MySQL > select * from TABLE(generate_series(start=>2, end=>5, step=>2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

对于输入行 (NULL, 10)，由于存在 NULL 值，因此不返回任何行。

## 关键词

表函数，生成序列
