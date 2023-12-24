---
displayed_sidebar: English
---

# generate_series

## 描述

生成一个值系列，该系列的起始值和结束值由 `start` 和 `end` 指定，并且可以指定步长 `step`。

generate_series() 是一个表函数。表函数可以为每个输入行返回一个行集。行集可以包含零行、一行或多行。每行可以包含一个或多个列。

在 StarRocks 中使用 generate_series()，如果输入参数是常量，则必须在 TABLE 关键字中使用。如果输入参数是表达式（例如列名），则不需要使用 TABLE 关键字（参见示例 5）。

此函数从 v3.1 开始支持。

## 语法

```SQL
generate_series(start, end [,step])
```

## 参数

- `start`：系列的起始值，必填。支持的数据类型为 INT、BIGINT 和 LARGEINT。
- `end`：系列的结束值，必填。支持的数据类型为 INT、BIGINT 和 LARGEINT。
- `step`：递增或递减的值，可选。支持的数据类型为 INT、BIGINT 和 LARGEINT。如果未指定，步长默认为 1。`step` 可以是负数或正数，但不能为零。

这三个参数的数据类型必须相同，例如 `generate_series(INT start, INT end [, INT step])`。
从 v3.3 开始支持命名参数，所有参数都使用名称输入，例如 `generate_series(start=>3, end=>7, step=>2)`。

## 返回值

返回一个值系列，该系列的起始值和结束值与输入参数 `start` 和 `end` 相同。

- 当 `step` 为正数时，如果 `start` 大于 `end`，则返回零行。相反，当 `step` 为负数时，如果 `start` 小于 `end`，则返回零行。
- 如果 `step` 为 0，则返回错误。
- 此函数处理 null 值如下：如果任何输入参数为字面上的 null，则报告错误。如果任何输入参数是表达式，并且表达式的结果为 null，则返回 0 行（参见示例 5）。

## 例子

示例 1：使用默认步长按升序在范围 [2,5] 内生成值序列。

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

示例 2：使用指定步长按升序在范围 [2,5] 内生成值序列。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, 2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

示例 3：使用指定步长按降序在范围 [5,2] 内生成值序列。

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

示例 4：当 `step` 为负数且 `start` 小于 `end` 时，返回零行。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, -1));
Empty set (0.01 sec)
```

示例 5：使用表列作为 generate_series() 的输入参数。在此用例中，您无需在 generate_series() 中使用 `TABLE()`。

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

-- 为行 (1,3) 和 (4,7) 生成步长为 1 的多行。
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

-- 为行 (5,2) 和 (9,6) 生成步长为 -1 的多行。
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
示例 6：使用命名参数按升序在范围 [2,5] 内生成值序列，步长为 2。

```SQL
MySQL > select * from TABLE(generate_series(start=>2, end=>5, step=>2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

输入行 `(NULL, 10)` 的值为 NULL，对于此行返回零行。

## 关键字

表函数，生成系列
