---
displayed_sidebar: "Chinese"
---

# generate_series

## 描述

在由`start`和`end`指定的区间内生成一系列值，并且带有一个可选的`step`。

generate_series()是一个表函数。表函数可以为每个输入行返回一个行集。行集可以包含零行、一行或多行。每一行可以包含一个或多个列。

要在StarRocks中使用generate_series()，如果输入参数是常量，则必须将其封装在`TABLE`关键字中。如果输入参数是表达式，例如列名，则不需要`TABLE`关键字（请参阅示例5）。

此函数从v3.1开始受支持。

## 语法

```SQL
generate_series(start, end [,step])
```

## 参数

- `start`：系列的起始值，必填。支持的数据类型为INT、BIGINT和LARGEINT。
- `end`：系列的结束值，必填。支持的数据类型为INT、BIGINT和LARGEINT。
- `step`：要增加或减少的值，可选。支持的数据类型为INT、BIGINT和LARGEINT。如果不指定，则默认步长为1。`step`可以是负值或正值，但不能为零。

这三个参数必须具有相同的数据类型，例如`generate_series(INT start, INT end [, INT step])`。

## 返回值

返回具有与输入参数`start`和`end`相同的一系列值。

- 当`step`为正时，如果`start`大于`end`，则返回零行。相反，当`step`为负时，如果`start`小于`end`，则返回零行。
- 如果`step`为0，则返回错误。
- 此函数处理null值如下：如果任何输入参数是字面上的null，则报告错误。如果任何输入参数是表达式，并且表达式的结果为null，则返回0行（请参阅示例5）。

## 示例

示例1：以默认步长`1`按升序方式生成范围[2,5]内的值序列。

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

示例2：以指定步长`2`按升序方式生成范围[2,5]内的值序列。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, 2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

示例3：以指定步长`-1`按降序方式生成范围[5,2]内的值序列。

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

示例4：当`step`为负且`start`小于`end`时返回零行。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, -1));
Empty set (0.01 sec)
```

示例5：使用表列作为generate_series()的输入参数。在此案例中，您无需在generate_series()中使用`TABLE()`。

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

-- 为行(1,3)和(4,7)生成具有步长1的多行。
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

-- 为行(5,2)和(9,6)生成具有步长-1的多行。
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

输入行`(NULL, 10)`具有空值，因此此行返回零行。

## 关键词

表函数, 生成系列