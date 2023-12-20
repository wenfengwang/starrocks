---
displayed_sidebar: English
---

# 窗口函数

## 背景

窗口函数是一类特殊的内置函数。它与聚合函数相似，都是对多个输入行进行计算以得出单个数据值。不同之处在于，窗口函数在特定的窗口内处理输入数据，而不是采用“group by”方法。每个窗口中的数据可以使用over()子句进行排序和分组。窗口函数**为每一行计算一个独立的值**，而不是为每个分组计算一个值。这种灵活性允许用户向select子句添加额外的列，并进一步筛选结果集。窗口函数只能出现在select列表和子句的最外层位置。它在查询的最后阶段生效，即在`join`、`where`和`group by`操作之后。窗口函数常用于分析趋势、计算离群值，以及在大规模数据上执行分桶分析。

## 用法

窗口函数的语法如下：

```SQL
function(args) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
partition_by_clause ::= PARTITION BY expr [, expr ...]
order_by_clause ::= ORDER BY expr [ASC | DESC] [, expr [ASC | DESC] ...]
```

### 函数

目前支持的函数包括：

* MIN()、MAX()、COUNT()、SUM()、AVG()
* FIRST_VALUE()、LAST_VALUE()、LEAD()、LAG()
* ROW_NUMBER()、RANK()、DENSE_RANK()
* CUME_DIST()、PERCENT_RANK()、QUALIFY()
* NTILE()
* VARIANCE()、VAR_SAMP()、STD()、STDDEV_SAMP()、COVAR_SAMP()、COVAR_POP()、CORR()

### PARTITION BY子句

Partition By子句类似于Group By。它根据一个或多个指定列对输入行进行分组。相同值的行被组合在一起。

### ORDER BY子句

Order By子句基本上与外部Order By相同。它定义了输入行的顺序。如果指定了Partition By，那么Order By定义了每个分区组内的顺序。唯一的区别是，OVER子句中的Order By n（n是正整数）相当于无操作，而外部Order By中的n表示按第n列排序。

例子：

此示例展示了如何向select列表添加一个id列，其值为1、2、3等，并根据events表中的date_and_time列进行排序。

```SQL
SELECT row_number() OVER (ORDER BY date_and_time) AS id,
    c1, c2, c3, c4
FROM events;
```

### 窗口子句

窗口子句用于指定行的操作范围（基于当前行的前行和后行）。它支持以下语法：AVG()、COUNT()、FIRST_VALUE()、LAST_VALUE()和SUM()。对于MAX()和MIN()，窗口子句可以指定从UNBOUNDED PRECEDING开始的范围。

语法：

```SQL
ROWS BETWEEN [ { m | UNBOUNDED } PRECEDING | CURRENT ROW] [ AND [CURRENT ROW | { UNBOUNDED | n } FOLLOWING] ]
```

示例：

假设我们有以下股票数据，股票代码为JDR，收盘价为每日收盘价。

```SQL
create table stock_ticker (
    stock_symbol string,
    closing_price decimal(8,2),
    closing_date timestamp);

-- ...load some data...

select *
from stock_ticker
order by stock_symbol, closing_date
```

原始数据显示如下：

```plaintext
+--------------+---------------+---------------------+
| stock_symbol | closing_price | closing_date        |
+--------------+---------------+---------------------+
| JDR          | 12.86         | 2014-10-02 00:00:00 |
| JDR          | 12.89         | 2014-10-03 00:00:00 |
| JDR          | 12.94         | 2014-10-04 00:00:00 |
| JDR          | 12.55         | 2014-10-05 00:00:00 |
| JDR          | 14.03         | 2014-10-06 00:00:00 |
| JDR          | 14.75         | 2014-10-07 00:00:00 |
| JDR          | 13.98         | 2014-10-08 00:00:00 |
+--------------+---------------+---------------------+
```

此查询使用窗口函数生成moving_average列，该列的值是3天（前一天、当天和次日）平均股价。第一天没有前一天的值，最后一天没有次日的值，因此这两行只计算了两天的平均值。这里Partition By不生效，因为所有数据都是JDR的数据。但是，如果有其他股票信息，Partition By将确保窗口函数在每个分区内操作。

```SQL
select stock_symbol, closing_date, closing_price,
    avg(closing_price)
        over (partition by stock_symbol
              order by closing_date
              rows between 1 preceding and 1 following
        ) as moving_average
from stock_ticker;
```

获得以下数据：

```plaintext
+--------------+---------------------+---------------+----------------+
| stock_symbol | closing_date        | closing_price | moving_average |
+--------------+---------------------+---------------+----------------+
| JDR          | 2014-10-02 00:00:00 | 12.86         | 12.87          |
| JDR          | 2014-10-03 00:00:00 | 12.89         | 12.89          |
| JDR          | 2014-10-04 00:00:00 | 12.94         | 12.79          |
| JDR          | 2014-10-05 00:00:00 | 12.55         | 13.17          |
| JDR          | 2014-10-06 00:00:00 | 14.03         | 13.77          |
| JDR          | 2014-10-07 00:00:00 | 14.75         | 14.25          |
| JDR          | 2014-10-08 00:00:00 | 13.98         | 14.36          |
+--------------+---------------------+---------------+----------------+
```

## 函数示例

本节介绍StarRocks支持的窗口函数。

### AVG()

语法：

```SQL
AVG(expr) [OVER (*analytic_clause*)]
```

示例：

计算当前行及其前后每一行的x的平均值。

```SQL
select x, property,
    avg(x)
        over (
            partition by property
            order by x
            rows between 1 preceding and 1 following
        ) as 'moving average'
from int_t
where property in ('odd','even');
```

```plaintext
+----+----------+----------------+
| x  | property | moving average |
+----+----------+----------------+
| 2  | even     | 3              |
| 4  | even     | 4              |
| 6  | even     | 6              |
| 8  | even     | 8              |
| 10 | even     | 9              |
| 1  | odd      | 2              |
| 3  | odd      | 3              |
| 5  | odd      | 5              |
| 7  | odd      | 7              |
| 9  | odd      | 8              |
+----+----------+----------------+
```

### COUNT()

语法：

```SQL
COUNT(expr) [OVER (analytic_clause)]
```

示例：

从当前行到第一行计数x的出现次数。

```SQL
select x, property,
    count(x)
        over (
            partition by property
            order by x
            rows between unbounded preceding and current row
        ) as 'cumulative total'
from int_t where property in ('odd','even');
```

```plaintext
+----+----------+------------------+
| x  | property | cumulative count |
+----+----------+------------------+
| 2  | even     | 1                |
| 4  | even     | 2                |
| 6  | even     | 3                |
| 8  | even     | 4                |
| 10 | even     | 5                |
| 1  | odd      | 1                |
| 3  | odd      | 2                |
| 5  | odd      | 3                |
| 7  | odd      | 4                |
| 9  | odd      | 5                |
+----+----------+------------------+
```

### CUME_DIST()

CUME_DIST()函数计算分区内某个值的累积分布，指示其作为小于或等于当前行中值的百分比的相对位置。其范围从0到1，适用于百分位计算和数据分布分析。

语法：

```SQL
CUME_DIST() OVER (partition_by_clause order_by_clause)
```

**此函数应与ORDER BY一起使用，以便按照所需顺序对分区行进行排序。如果没有ORDER BY，所有行都是同等的，其值为N/N = 1，其中N是分区大小。**

CUME_DIST()包含NULL值，并将它们视为最低值。

以下示例展示了y列在x列的每个分组内的累积分布。

```SQL
SELECT x, y,
    CUME_DIST()
        OVER (
            PARTITION BY x
            ORDER BY y
        ) AS `cume_dist`
FROM int_t;
```

```plaintext
+---+---+--------------------+
| x | y | cume_dist          |
+---+---+--------------------+
| 1 | 1 | 0.3333333333333333 |
| 1 | 2 |                  1 |
| 1 | 2 |                  1 |
| 2 | 1 | 0.3333333333333333 |
| 2 | 2 | 0.6666666666666667 |
| 2 | 3 |                  1 |
| 3 | 1 | 0.6666666666666667 |
| 3 | 1 | 0.6666666666666667 |
| 3 | 2 |                  1 |
+---+---+--------------------+
```

### DENSE_RANK()

DENSE_RANK()函数用于表示排名。与RANK()不同，DENSE_RANK()**没有空缺**的数字。例如，如果有两个并列的1，则DENSE_RANK()的第三个数字仍是2，而RANK()的第三个数字是3。

语法：

```SQL
DENSE_RANK() OVER(partition_by_clause order_by_clause)
```

以下示例展示了根据属性列分组的x列的排名。

```SQL
select x, y,
    dense_rank()
        over (
            partition by x
            order by y
        ) as `rank`
from int_t;
```

```plaintext
+---+---+------+
| x | y | rank |
+---+---+------+
| 1 | 1 | 1    |
| 1 | 2 | 2    |
| 1 | 2 | 2    |
| 2 | 1 | 1    |
| 2 | 2 | 2    |
| 2 | 3 | 3    |
| 3 | 1 | 1    |
| 3 | 1 | 1    |
| 3 | 2 | 2    |
+---+---+------+
```

### NTILE()

NTILE()函数将分区中的已排序行按指定的num_buckets数量尽可能均等地划分，将划分的行存储在各自的桶中，从1开始编号[1, 2, ..., num_buckets]，并返回每行所在的桶编号。

关于桶大小：

* 如果行数可以被指定的num_buckets数量精确除尽，则所有桶的大小相同。
* 如果行数不能被指定的num_buckets数量精确除尽，则会有两种不同大小的桶。大小之间的差异为1。拥有更多行的桶将排在行数较少的桶前面。

语法：

```SQL
NTILE (num_buckets) OVER (partition_by_clause order_by_clause)
```

num_buckets：要创建的桶的数量。该值必须是一个常数正整数，最大值为2^63 - 1。

NTILE()函数中不允许使用窗口子句。

NTILE()函数返回BIGINT类型的数据。

示例：

以下示例将分区中的所有行分成2个桶。

```sql
select id, x, y,
    ntile(2)
        over (
            partition by x
            order by y
        ) as bucket_id
from t1;
```

```plaintext
+------+------+------+-----------+
| id   | x    | y    | bucket_id |
+------+------+------+-----------+
|    1 |    1 |   11 |         1 |
|    2 |    1 |   11 |         1 |
|    3 |    1 |   22 |         1 |
|    4 |    1 |   33 |         2 |
|    5 |    1 |   44 |         2 |
|    6 |    1 |   55 |         2 |
|    7 |    2 |   66 |         1 |
|    8 |    2 |   77 |         1 |
|    9 |    2 |   88 |         2 |
|   10 |    3 |   99 |         1 |
+------+------+------+-----------+
```

如上例所示，当num_buckets为2时：

* 编号1到6的行被分类到第一个分区；编号1到3的行存储在第一个桶中，编号4到6的行存储在第二个桶中。
* 编号7到9的行被分类到第二个分区；编号7和8的行存储在第一个桶中，编号9的行存储在第二个桶中。
* 编号10的行被分类到第三个分区并存储在第一个桶中。

<br/>


### FIRST_VALUE()

FIRST_VALUE()返回窗口范围内的**第一个**值。

语法：

```SQL
FIRST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

从v2.5.0版本开始支持IGNORE NULLS。它用于确定是否从计算中排除expr的NULL值。默认情况下，包括NULL值，这意味着如果过滤结果中的第一个值是NULL，则返回NULL。如果指定了IGNORE NULLS，则返回过滤结果中的第一个非NULL值。如果所有值都是NULL，即使指定了IGNORE NULLS，也会返回NULL。

示例：

我们有以下数据：

```SQL
 select name, country, greeting
 from mail_merge;
```

```plaintext
+---------+---------+--------------+
| name    | country | greeting     |
+---------+---------+--------------+
| Pete    | USA     | Hello        |
| John    | USA     | Hi           |
| Boris   | Germany | Guten tag    |
| Michael | Germany | Guten morgen |
| Bjorn   | Sweden  | Hej          |
| Mats    | Sweden  | Tja          |
+---------+---------+--------------+
```

使用FIRST_VALUE()根据国家分组返回每组中的第一个问候语值。

```SQL
select country, name,
    first_value(greeting)
        over (
            partition by country
            order by name, greeting
        ) as greeting
from mail_merge;
```

```plaintext
+---------+---------+-----------+
| country | name    | greeting  |
+---------+---------+-----------+
| Germany | Boris   | Guten tag |
| Germany | Michael | Guten tag |
| Sweden  | Bjorn   | Hej       |
| Sweden  | Mats    | Hej       |
| USA     | John    | Hi        |
| USA     | Pete    | Hi        |
+---------+---------+-----------+
```

### LAG()

返回当前行之前offset行的行的值。此函数通常用于比较行之间的值和筛选数据。

LAG()可以用来查询以下类型的数据：

* 数字：TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL
* 字符串：CHAR、VARCHAR
* 日期：DATE、DATETIME
* 从StarRocks v2.5版本开始支持BITMAP和HLL。

语法：

```SQL
LAG(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

参数：

* expr：您要计算的字段。
* `offset`：偏移量。它必须是一个**正整数**。若未指定此参数，默认为1。
* default：如果找不到匹配的行，则返回的默认值。若未指定此参数，默认为NULL。default支持任何与expr类型兼容的表达式。
* 从v3.0版本开始支持IGNORE NULLS。它用于确定是否在结果中包含expr的NULL值。默认情况下，计算偏移行时包含NULL值，这意味着如果目标行的值是NULL，则返回NULL。参见示例1。如果指定了IGNORE NULLS，则在计算偏移行时忽略NULL值，并继续搜索偏移的非NULL值。如果找不到偏移的非NULL值，则返回NULL或default（如果指定）。参见示例2。

示例1：未指定IGNORE NULLS

创建一个表并插入值：

```SQL
CREATE TABLE test_tbl (col_1 INT, col_2 INT)
DISTRIBUTED BY HASH(col_1);

INSERT INTO test_tbl VALUES 
    (1, NULL),
    (2, 4),
    (3, NULL),
    (4, 2),
    (5, NULL),
    (6, 7),
    (7, 6),
    (8, 5),
    (9, NULL),
    (10, NULL);
```

从此表中查询数据，其中offset为2，表示遍历前两行；default为0，意味着如果没有找到匹配的行，则返回0。

输出：

```plaintext
SELECT col_1, col_2, LAG(col_2,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+---------------------------------------------+
| col_1 | col_2 | lag(col_2, 2, 0) OVER (ORDER BY col_1 ASC ) |
+-------+-------+---------------------------------------------+
|     1 |  NULL |                                           0 |
|     2 |     4 |                                           0 |
|     3 |  NULL |                                        NULL |
|     4 |     2 |                                           4 |
|     5 |  NULL |                                        NULL |
|     6 |     7 |                                           2 |
|     7 |     6 |                                        NULL |
|     8 |     5 |                                           7 |
|     9 |  NULL |                                           6 |
|    10 |  NULL |                                           5 |
+-------+-------+---------------------------------------------+
```

对于前两行，不存在前两行，返回默认值0。

对于第3行中的NULL，向后两行的值是NULL，返回NULL，因为允许NULL值。

示例2：指定了IGNORE NULLS

使用上述表和参数设置。

```SQL
SELECT col_1, col_2, LAG(col_2 IGNORE NULLS,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+---------------------------------------------+
| col_1 | col_2 | lag(col_2, 2, 0) OVER (ORDER BY col_1 ASC ) |
+-------+-------+---------------------------------------------+
|     1 |  NULL |                                           0 |
|     2 |     4 |                                           0 |
|     3 |  NULL |                                           0 |
|     4 |     2 |                                           0 |
|     5 |  NULL |                                           4 |
|     6 |     7 |                                           4 |
|     7 |     6 |                                           2 |
|     8 |     5 |                                           7 |
|     9 |  NULL |                                           6 |
|    10 |  NULL |                                           6 |
+-------+-------+---------------------------------------------+
```

对于第1行到第4行，系统在之前的行中找不到两个非NULL值，因此返回默认值0。

对于第7行中的值6，向后两行的值是NULL，但由于指定了IGNORE NULLS，因此忽略了NULL。系统继续搜索非NULL值，并返回第4行中的2。

### LAST_VALUE()

LAST_VALUE()返回窗口范围的**最后**一个值。它与FIRST_VALUE()相对应。

语法：

```SQL
LAST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

从v2.5.0版本开始支持IGNORE NULLS。它用于确定是否从计算中排除expr的NULL值。默认情况下，包括NULL值，这意味着如果过滤结果中的最后一个值是NULL，则返回NULL。如果指定了IGNORE NULLS，则返回过滤结果中的最后一个非NULL值。如果所有值都是NULL，即使指定了IGNORE NULLS，也会返回NULL。

使用示例中的数据：

```SQL
select country, name,
    last_value(greeting)
        over (
            partition by country
            order by name, greeting
        ) as greeting
from mail_merge;
```

```plaintext
+---------+---------+--------------+
| country | name    | greeting     |
+---------+---------+--------------+
| Germany | Boris   | Guten morgen |
| Germany | Michael | Guten morgen |
| Sweden  | Bjorn   | Tja          |
| Sweden  | Mats    | Tja          |
| USA     | John    | Hello        |
| USA     | Pete    | Hello        |
+---------+---------+--------------+
```

### LEAD()

返回当前行之后offset行的行的值。此函数通常用于比较行之间的值和筛选数据。

`LEAD()` 函数可以查询与 [LAG()](#lag) 函数支持的数据类型相同的数据。

语法：

```sql
LEAD(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

参数：

* expr：您想要计算的字段。
* offset：偏移量。它必须是正整数。如果未指定此参数，默认值为 1。
* default：如果找不到匹配的行，则返回的默认值。如果未指定此参数，默认值为 NULL。default 支持任何与 expr 类型兼容的表达式。
* 从 v3.0 版本开始支持 IGNORE NULLS。它用于确定结果中是否包括 expr 的 NULL 值。默认情况下，在计算偏移行时会包括 NULL 值，这意味着如果目标行的值为 NULL，则返回 NULL。请参见示例 1。如果指定了 IGNORE NULLS，那么在计数偏移行时会忽略 NULL 值，系统将继续寻找偏移的非空值。如果找不到偏移的非空值，则返回 NULL 或 default（如果指定了的话）。请参见示例 2。

示例 1：未指定 IGNORE NULLS

创建表并插入值：

```SQL
CREATE TABLE test_tbl (col_1 INT, col_2 INT)
DISTRIBUTED BY HASH(col_1);

INSERT INTO test_tbl VALUES 
    (1, NULL),
    (2, 4),
    (3, NULL),
    (4, 2),
    (5, NULL),
    (6, 7),
    (7, 6),
    (8, 5),
    (9, NULL),
    (10, NULL);
```

查询此表中的数据，其中 offset 为 2，表示查找后两行的数据；default 为 0，意味着如果找不到匹配的行，则返回 0。

输出：

```plaintext
SELECT col_1, col_2, LEAD(col_2,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+----------------------------------------------+
| col_1 | col_2 | lead(col_2, 2, 0) OVER (ORDER BY col_1 ASC ) |
+-------+-------+----------------------------------------------+
|     1 |  NULL |                                         NULL |
|     2 |     4 |                                            2 |
|     3 |  NULL |                                         NULL |
|     4 |     2 |                                            7 |
|     5 |  NULL |                                            6 |
|     6 |     7 |                                            5 |
|     7 |     6 |                                         NULL |
|     8 |     5 |                                         NULL |
|     9 |  NULL |                                            0 |
|    10 |  NULL |                                            0 |
+-------+-------+----------------------------------------------+
```

对于第一行，由于允许 NULL 值，向前两行的值是 NULL，因此返回 NULL。

对于最后两行，由于不存在后续两行，因此返回默认值 0。

示例 2：指定了 IGNORE NULLS

使用上述表和参数设置。

```SQL
SELECT col_1, col_2, LEAD(col_2 IGNORE NULLS,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+----------------------------------------------+
| col_1 | col_2 | lead(col_2, 2, 0) OVER (ORDER BY col_1 ASC ) |
+-------+-------+----------------------------------------------+
|     1 |  NULL |                                            2 |
|     2 |     4 |                                            7 |
|     3 |  NULL |                                            7 |
|     4 |     2 |                                            6 |
|     5 |  NULL |                                            6 |
|     6 |     7 |                                            5 |
|     7 |     6 |                                            0 |
|     8 |     5 |                                            0 |
|     9 |  NULL |                                            0 |
|    10 |  NULL |                                            0 |
+-------+-------+----------------------------------------------+
```

对于第 7 行到第 10 行，系统在后续行中找不到两个非空值，因此返回默认值 0。

对于第一行，向前两行的值是 NULL，由于指定了 IGNORE NULLS，因此忽略了 NULL。系统继续搜索第二个非空值，返回第 4 行中的 2。

### MAX()

返回当前窗口中指定行的最大值。

语法：

```SQL
MAX(expr) [OVER (analytic_clause)]
```

示例：

计算从第一行到当前行之后的行的最大值。

```SQL
select x, property,
    max(x)
        over (
            order by property, x
            rows between unbounded preceding and 1 following
        ) as 'local maximum'
from int_t
where property in ('prime','square');
```

```plaintext
+---+----------+---------------+
| x | property | local maximum |
+---+----------+---------------+
| 2 | prime    | 3             |
| 3 | prime    | 5             |
| 5 | prime    | 7             |
| 7 | prime    | 7             |
| 1 | square   | 7             |
| 4 | square   | 9             |
| 9 | square   | 9             |
+---+----------+---------------+
```

从 StarRocks 2.4 版本开始，您可以将行范围指定为从当前行的前 n 行到后 n 行。

示例语句：

```sql
select x, property,
    max(x)
        over (
            order by property, x
            rows between 3 preceding and 2 following) as 'local maximum'
from int_t
where property in ('prime','square');
```

### MIN()

返回当前窗口中指定行的最小值。

语法：

```SQL
MIN(expr) [OVER (analytic_clause)]
```

示例：

计算从第一行到当前行之后的行的最小值。

```SQL
select x, property,
    min(x)
        over (
            order by property, x desc
            rows between unbounded preceding and 1 following
        ) as 'local minimum'
from int_t
where property in ('prime','square');
```

```plaintext
+---+----------+---------------+
| x | property | local minimum |
+---+----------+---------------+
| 7 | prime    | 5             |
| 5 | prime    | 3             |
| 3 | prime    | 2             |
| 2 | prime    | 2             |
| 9 | square   | 2             |
| 4 | square   | 1             |
| 1 | square   | 1             |
+---+----------+---------------+
```

从 StarRocks 2.4 版本开始，您可以将行范围指定为从当前行的前 n 行到后 n 行。

示例语句：

```sql
select x, property,
    min(x)
    over (
          order by property, x desc
          rows between 3 preceding and 2 following) as 'local minimum'
from int_t
where property in ('prime','square');
```

### PERCENT_RANK()

PERCENT_RANK() 函数计算结果集中行的相对排名，并以百分比形式返回。它返回小于当前行值的分区值的百分比，不包括最高值。返回值的范围是 0 到 1。此函数适用于百分位计算和数据分布分析。

PERCENT_RANK() 函数使用下面的公式计算，其中 rank 表示行排名，rows 表示分区行数：

```plaintext
(rank - 1) / (rows - 1)
```

语法：

```SQL
PERCENT_RANK() OVER (partition_by_clause order_by_clause)
```

**此函数应当与 ORDER BY 一起使用，以便按照所需的顺序对分区行进行排序。如果没有 ORDER BY，所有行都是同等的，并且值为 (1 - 1)/(N - 1) = 0，其中 N 是分区大小。**

以下示例显示了在 x 列的每个分组内 y 列的相对排名。

```SQL
SELECT x, y,
    PERCENT_RANK()
        OVER (
            PARTITION BY x
            ORDER BY y
        ) AS `percent_rank`
FROM int_t;
```

```plaintext
+---+---+--------------+
| x | y | percent_rank |
+---+---+--------------+
| 1 | 1 |            0 |
| 1 | 2 |          0.5 |
| 1 | 2 |          0.5 |
| 2 | 1 |            0 |
| 2 | 2 |          0.5 |
| 2 | 3 |            1 |
| 3 | 1 |            0 |
| 3 | 1 |            0 |
| 3 | 2 |            1 |
+---+---+--------------+
```

### RANK()

RANK() 函数用于表示排名。与 DENSE_RANK() 不同，RANK() 会**出现空缺的**编号。例如，如果有两个并列的第 1 名，那么 RANK() 的第三个编号将是 3 而不是 2。

语法：

```SQL
RANK() OVER(partition_by_clause order_by_clause)
```

示例：

根据 x 列的属性进行排名：

```SQL
select x, y, rank() over(partition by x order by y) as `rank`
from int_t;
```

```plaintext
+---+---+------+
| x | y | rank |
+---+---+------+
| 1 | 1 | 1    |
| 1 | 2 | 2    |
| 1 | 2 | 2    |
| 2 | 1 | 1    |
| 2 | 2 | 2    |
| 2 | 3 | 3    |
| 3 | 1 | 1    |
| 3 | 1 | 1    |
| 3 | 2 | 3    |
+---+---+------+
```

### ROW_NUMBER()

为分区中的每一行返回一个从 1 开始的连续递增整数。与 RANK() 和 DENSE_RANK() 不同，ROW_NUMBER() 返回的值**不会重复或留有空缺**，而是**连续递增**。

语法：

```SQL
ROW_NUMBER() OVER(partition_by_clause order_by_clause)
```

示例：

```SQL
select x, y, row_number() over(partition by x order by y) as `rank`
from int_t;
```

```plaintext
+---+---+------+
| x | y | rank |
+---+---+------+
| 1 | 1 | 1    |
| 1 | 2 | 2    |
| 1 | 2 | 3    |
| 2 | 1 | 1    |
| 2 | 2 | 2    |
| 2 | 3 | 3    |
| 3 | 1 | 1    |
| 3 | 1 | 2    |
| 3 | 2 | 3    |
+---+---+------+
```

### QUALIFY()

QUALIFY 子句用于过滤窗口函数的结果。在 SELECT 语句中，您可以使用 QUALIFY 子句对列应用条件以过滤结果。QUALIFY 类似于聚合函数中的 HAVING 子句。该功能从 v2.5 版本开始支持。

QUALIFY 简化了 SELECT 语句的编写。

在使用 QUALIFY 之前，SELECT 语句可能如下所示：

```SQL
SELECT *
FROM (SELECT DATE,
             PROVINCE_CODE,
             TOTAL_SCORE,
             ROW_NUMBER() OVER(PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) AS SCORE_ROWNUMBER
      FROM example_table) T1
WHERE T1.SCORE_ROWNUMBER = 1;
```

使用 QUALIFY 后，语句缩短为：

```SQL
SELECT DATE, PROVINCE_CODE, TOTAL_SCORE
FROM example_table 
QUALIFY ROW_NUMBER() OVER(PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) = 1;
```

QUALIFY 仅支持以下三个窗口函数：ROW_NUMBER()、RANK() 和 DENSE_RANK()。

**语法:**

```SQL
SELECT <column_list>
FROM <data_source>
[GROUP BY ...]
[HAVING ...]
QUALIFY <window_function>
[ ... ]
```

**参数：**

<column_list>：您想要获取数据的列。

<data_source>：数据源通常是一个表。

<window_function>：QUALIFY 子句后面只能跟随窗口函数，包括 ROW_NUMBER()、RANK() 和 DENSE_RANK()。

**示例:**

```SQL
-- Create a table.
CREATE TABLE sales_record (
   city_id INT,
   item STRING,
   sales INT
) DISTRIBUTED BY HASH(`city_id`);

-- Insert data into the table.
insert into sales_record values
(1,'fruit',95),
(2,'drinks',70),
(3,'fruit',87),
(4,'drinks',98);

-- Query data from the table.
select * from sales_record order by city_id;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       1 | fruit  |    95 |
|       2 | drinks |    70 |
|       3 | fruit  |    87 |
|       4 | drinks |    98 |
+---------+--------+-------+
```

示例 1：从表中获取行号大于 1 的记录。

```SQL
SELECT city_id, item, sales
FROM sales_record
QUALIFY row_number() OVER (ORDER BY city_id) > 1;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       2 | drinks |    70 |
|       3 | fruit  |    87 |
|       4 | drinks |    98 |
+---------+--------+-------+
```

示例 2：从表的每个分区中获取行号为 1 的记录。该表按项目分为两个分区，并返回每个分区中的第一行。

```SQL
SELECT city_id, item, sales
FROM sales_record 
QUALIFY ROW_NUMBER() OVER (PARTITION BY item ORDER BY city_id) = 1
ORDER BY city_id;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       1 | fruit  |    95 |
|       2 | drinks |    70 |
+---------+--------+-------+
2 rows in set (0.01 sec)
```

示例 3：从表的每个分区中获取销售排名第 1 的记录。该表按项目分为两个分区，并返回每个分区中销售额最高的行。

```SQL
SELECT city_id, item, sales
FROM sales_record
QUALIFY RANK() OVER (PARTITION BY item ORDER BY sales DESC) = 1
ORDER BY city_id;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       1 | fruit  |    95 |
|       4 | drinks |    98 |
+---------+--------+-------+
```

**使用注意：**

具有 QUALIFY 的查询中子句的执行顺序按以下顺序进行评估：

1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. WINDOW
6. QUALIFY
7. DISTINCT
8. ORDER BY
9. LIMIT

### SUM()

语法：

```SQL
SUM(expr) [OVER (analytic_clause)]
```

示例：

按属性分组，并计算组内**当前行，前一行和后一行**的总和。

```SQL
select x, property,
    sum(x)
        over (
            partition by property
            order by x
            rows between 1 preceding and 1 following
        ) as 'moving total'
from int_t where property in ('odd','even');
```

```plaintext
+----+----------+--------------+
| x  | property | moving total |
+----+----------+--------------+
| 2  | even     | 6            |
| 4  | even     | 12           |
| 6  | even     | 18           |
| 8  | even     | 24           |
| 10 | even     | 18           |
| 1  | odd      | 4            |
| 3  | odd      | 9            |
| 5  | odd      | 15           |
| 7  | odd      | 21           |
+----+----------+--------------+
```

### VARIANCE、VAR_POP、VARIANCE_POP

返回表达式的总体方差。VAR_POP 和 VARIANCE_POP 是 VARIANCE 的别名。从 v2.5.10 版本开始，这些函数可以作为窗口函数使用。

**语法:**

```SQL
VARIANCE(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，这个窗口函数支持 **ORDER BY** 和 **Window** 子句。

**参数：**

如果 expr 是表列，它的计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例:**

假设表 agg 有以下数据：

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

使用 VARIANCE() 函数。

```plaintext
mysql> select variance(k) over (partition by no) FROM agg;
+-------------------------------------+
| variance(k) OVER (PARTITION BY no ) |
+-------------------------------------+
|                                   0 |
|                             54.6875 |
|                             54.6875 |
|                             54.6875 |
|                             54.6875 |
+-------------------------------------+

mysql> select variance(k) over(
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+-------------------+
| window_test       |
+-------------------+
|                 0 |
|                25 |
| 38.88888888888889 |
|           54.6875 |
|           54.6875 |
+-------------------+
```

### VAR_SAMP、VARIANCE_SAMP

返回表达式的样本方差。从 v2.5.10 版本开始，这些函数可以作为窗口函数使用。

**语法:**

```sql
VAR_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，这个窗口函数支持 **ORDER BY** 和 **Window** 子句。

**参数：**

如果 expr 是表列，它的计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例：**

假设表 agg 有以下数据：

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

使用 VAR_SAMP() 窗口函数。

```plaintext
mysql> select VAR_SAMP(k) over (partition by no) FROM agg;
+-------------------------------------+
| var_samp(k) OVER (PARTITION BY no ) |
+-------------------------------------+
|                                   0 |
|                   72.91666666666667 |
|                   72.91666666666667 |
|                   72.91666666666667 |
|                   72.91666666666667 |
+-------------------------------------+

mysql> select VAR_SAMP(k) over(
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+--------------------+
| window_test        |
+--------------------+
|                  0 |
|                 50 |
| 58.333333333333336 |
|  72.91666666666667 |
|  72.91666666666667 |
+--------------------+
```

### STD、STDDEV、STDDEV_POP

返回表达式的标准差。从 v2.5.10 版本开始，这些函数可以作为窗口函数使用。

**语法:**

```sql
STD(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，这个窗口函数支持 **ORDER BY** 和 **Window** 子句。

**参数：**

如果 expr 是表列，它的计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例:**

假设表 agg 有以下数据：

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

使用 STD() 窗口函数。

```plaintext
mysql> select STD(k) over (partition by no) FROM agg;
+--------------------------------+
| std(k) OVER (PARTITION BY no ) |
+--------------------------------+
|                              0 |
|               7.39509972887452 |
|               7.39509972887452 |
|               7.39509972887452 |
|               7.39509972887452 |
+--------------------------------+

mysql> select std(k) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+-------------------+
| window_test       |
+-------------------+
|                 0 |
|                 5 |
| 6.236095644623236 |
|  7.39509972887452 |
|  7.39509972887452 |
+-------------------+
```

### STDDEV_SAMP

返回表达式的样本标准差。从 v2.5.10 版本开始，此函数可以作为窗口函数使用。

**语法:**

```sql
STDDEV_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，这个窗口函数支持 **ORDER BY** 和 **Window** 子句。

**参数：**

如果 expr 是表列，它的计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例:**

假设表 agg 有以下数据：

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

使用 STDDEV_SAMP() 窗口函数。

```plaintext
mysql> select STDDEV_SAMP(k) over (partition by no) FROM agg;
+----------------------------------------+
| stddev_samp(k) OVER (PARTITION BY no ) |
+----------------------------------------+
|                                      0 |
|                      8.539125638299666 |
|                      8.539125638299666 |
|                      8.539125638299666 |
|                      8.539125638299666 |
+----------------------------------------+

mysql> select STDDEV_SAMP(k) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+--------------------+
| window_test        |
+--------------------+
|                  0 |
| 7.0710678118654755 |
|  7.637626158259733 |
|  8.539125638299666 |
|  8.539125638299666 |
+--------------------+
```

### COVAR_SAMP

返回两个表达式的样本协方差。此函数从 v2.5.10 版本开始得到支持，并且它也是一个聚合函数。

**语法:**

```sql
COVAR_SAMP(expr1,expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，这个窗口函数支持 **ORDER BY** 和 **Window** 子句。

**参数：**

如果 expr 是表列，它的计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例**：

假设表 agg 有以下数据：

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

使用 COVAR_SAMP() 窗口函数。

```plaintext
mysql> select COVAR_SAMP(k, v) over (partition by no) FROM agg;
+------------------------------------------+
| covar_samp(k, v) OVER (PARTITION BY no ) |
+------------------------------------------+
|                                     NULL |
|                       119.99999999999999 |
|                       119.99999999999999 |
|                       119.99999999999999 |
|                       119.99999999999999 |
+------------------------------------------+

mysql> select COVAR_SAMP(k,v) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+--------------------+
| window_test        |
+--------------------+
|               NULL |
|                 55 |
|                 55 |
| 119.99999999999999 |
| 119.99999999999999 |
+--------------------+
```

### COVAR_POP

返回两个表达式的总体协方差。此函数从 v2.5.10 版本开始得到支持，并且它也是一个聚合函数。

**语法:**

```sql
COVAR_POP(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，这个窗口函数支持 **ORDER BY** 和 **Window** 子句。

**参数：**

如果 expr 是表列，它的计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例:**

假设表 agg 有以下数据：

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

使用 COVAR_POP() 窗口函数。

```plaintext
mysql> select COVAR_POP(k, v) over (partition by no) FROM agg;
+-----------------------------------------+
| covar_pop(k, v) OVER (PARTITION BY no ) |
+-----------------------------------------+
|                                    NULL |
|                       79.99999999999999 |
|                       79.99999999999999 |
|                       79.99999999999999 |
|                       79.99999999999999 |
+-----------------------------------------+

mysql> select COVAR_POP(k,v) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+-------------------+
| window_test       |
+-------------------+
|              NULL |
|              27.5 |
|              27.5 |
| 79.99999999999999 |
| 79.99999999999999 |
+-------------------+
```

### CORR

返回两个表达式之间的皮尔逊相关系数。此函数从 v2.5.10 版本开始得到支持，并且它也是一个聚合函数。

**语法:**

```sql
CORR(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，这个窗口函数支持 **ORDER BY** 和 **Window** 子句。

**参数：**

如果 expr 是表列，它的计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例:**

假设表 agg 有以下数据：

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

使用 CORR() 窗口函数。

```plaintext
mysql> select CORR(k, v) over (partition by no) FROM agg;
+------------------------------------+
| corr(k, v) OVER (PARTITION BY no ) |
+------------------------------------+
|                               NULL |
|                 0.9988445981121532 |
|                 0.9988445981121532 |
|                 0.9988445981121532 |
|                 0.9988445981121532 |
+------------------------------------+

mysql> select CORR(k,v) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+--------------------+
| window_test        |
+--------------------+
|               NULL |
|                  1 |
|                  1 |
| 0.9988445981121532 |
| 0.9988445981121532 |
+--------------------+
```
