---
displayed_sidebar: "Chinese"
---

# 窗口函数

## 背景

窗口函数是一种特殊类内建函数。它与聚合函数类似，也对多个输入行进行计算以获得单个数据值。不同之处在于，窗口函数处理输入数据在特定窗口内，而不是使用“group by”方法。每个窗口内的数据可以使用over()子句进行排序和分组。窗口函数**为每行计算一个单独的值**，而不是为每个组计算一个值。这种灵活性允许用户向select子句添加额外的列，并进一步过滤结果集。窗口函数只能出现在select列表和子句的最外层位置。它在查询结束时生效，即在执行`join`、`where`和`group by`操作之后。窗口函数通常用于分析趋势、计算离群值，并在大规模数据上执行分桶分析。

## 用法

窗口函数的语法：

```SQL
function(args) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
partition_by_clause ::= PARTITION BY expr [, expr ...]
order_by_clause ::= ORDER BY expr [ASC | DESC] [, expr [ASC | DESC] ...]
```

### 函数

当前支持的函数包括：

* MIN()，MAX()，COUNT()，SUM()，AVG()
* FIRST_VALUE()，LAST_VALUE()，LEAD()，LAG()
* ROW_NUMBER()，RANK()，DENSE_RANK()
* CUME_DIST()，PERCENT_RANK()，QUALIFY()
* NTILE()
* VARIANCE()，VAR_SAMP()，STD()，STDDEV_SAMP()，COVAR_SAMP()，COVAR_POP()，CORR()

### PARTITION BY 子句

Partition By子句类似于Group By。它根据一个或多个指定列对输入行进行分组。具有相同值的行被分组在一起。

### ORDER BY 子句

`Order By`子句基本上与外部`Order By`相同。它定义了输入行的顺序。如果指定了`Partition By`，`Order By`定义了每个分区分组内的顺序。唯一的区别在于`OVER`子句中的`Order By n`（n为正整数）等同于无操作，而外部`Order By`中的`n`表示按第n列排序。

示例：

此示例显示了向select列表添加id列，其值为1、2、3等，按events表中的`date_and_time`列排序。

```SQL
SELECT row_number() OVER (ORDER BY date_and_time) AS id,
    c1, c2, c3, c4
FROM events;
```

### Window子句

窗口子句用于指定操作的一系行（基于当前行的前导和跟随行）。它支持以下语法：AVG()，COUNT()，FIRST_VALUE()，LAST_VALUE()和SUM()。对于MAX()和MIN()，窗口子句可以指定开始为`UNBOUNDED PRECEDING`。

语法：

```SQL
ROWS BETWEEN [ { m | UNBOUNDED } PRECEDING | CURRENT ROW] [ AND [CURRENT ROW | { UNBOUNDED | n } FOLLOWING] ]
```

示例：

假设我们有以下股票数据，股票符号为JDR，收盘价格为每日收盘价格。

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

原始数据如下所示：

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

此查询使用窗口函数生成移动平均列，其值为3天（前一天、当天和后一天）的平均股票价格。第一天没有其前一天的值，最后一天没有后一天的值，因此这两行仅计算两天的平均值。这里`Partition By`不生效，因为所有数据都是JDR数据。但是，如果有其他股票信息，`Partition By`将确保窗口函数在每个分区内操作。

```SQL
select stock_symbol, closing_date, closing_price,
    avg(closing_price)
        over (partition by stock_symbol
              order by closing_date
              rows between 1 preceding and 1 following
        ) as moving_average
from stock_ticker;
```

得到如下数据：

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

本节描述了StarRocks支持的窗口函数。

### AVG()

语法：

```SQL
AVG(expr) [OVER (*analytic_clause*)]
```

示例：

计算当前行以及之前和之后每行的x平均值。

```SQL
select x, property,
    avg(x)
        over (
            partition by property
            order by x
            rows between 1 preceding and 1 following
        ) as '移动平均值'
from int_t
where property in ('odd','even');
```

```plaintext
+----+----------+----------------+
| x  | property | 移动平均值     |
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

从当前行到第一行计算x的出现次数。

```SQL
select x, property,
    count(x)
        over (
            partition by property
            order by x
            rows between unbounded preceding and current row
        ) as '累计总数'
from int_t where property in ('odd','even');
```

```plaintext
+----+----------+------------------+
| x  | property | 累计计数         |
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

CUME_DIST()函数计算在分区内一个值的累积分布，并指示其相对位置作为当前行中值小于等于该值的百分比。它的范围为0到1，对于百分位数计算和数据分布分析非常有用。

语法：

```SQL
CUME_DIST() OVER (partition_by_clause order_by_clause)
```

**应该使用ORDER BY来对分区行进行排序。没有ORDER BY时，所有行都是同等级别的，并且值为N/N = 1，其中N是分区大小。**

CUME_DIST()包含NULL值并将其视为最低值。

以下示例显示了列y在列x的每个分组中的累积分布。

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

DENSE_RANK()函数用于表示排名。与RANK()不同，DENSE_RANK()**不会留有**空缺的数字。例如，如果有两个并列的1，DENSE_RANK()的第三个数字仍然是2，而RANK()的第三个数字是3。

语法：

```SQL
DENSE_RANK() OVER(partition_by_clause order_by_clause)
```

以下示例显示了根据属性列分组的列x的排名。

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

NTILE()函数将分区中排序的行按指定数量的 `num_buckets` 尽可能均匀地划分，将划分的行存储在相应的桶中，从1开始 `[1, 2, ..., num_buckets]`，并返回每行所在的桶编号。

关于桶的大小：

* 如果行数能够被指定的 `num_buckets` 完全整除，所有桶的大小将相同。
* 如果行数不能被指定的 `num_buckets` 完全整除，将会有两个不同大小的桶。大小之间的差异为1。拥有更多行的桶将在行数较少的桶之前列出。

语法：

```SQL
NTILE (num_buckets) OVER (partition_by_clause order_by_clause)
```

`num_buckets`：要创建的桶的数量。该值必须是常量的正整数，其最大值为`2^63 - 1`。

NTILE()函数中不允许窗口子句。

NTILE()函数返回BIGINT类型的数据。

示例：

以下示例将分区中的所有行划分为2个桶。

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

正如上面的示例所示，当 `num_buckets` 为 `2` 时：

* 第1到第6行被分类到第一个分区; 第1到第3行存储在第一个桶中，第4到第6行存储在第二个桶中。
* 第7到第9行被分类到第二个分区; 第7和第8行存储在第一个桶中，第9行存储在第二个桶中。
* 第10行被分类到第三个分区，并存储在第一个桶中。

### FIRST_VALUE()

FIRST_VALUE()返回窗口范围内的**第一个**值。

语法：

```SQL

FIRST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```


`IGNORE NULLS` 支持从v2.5.0开始。其用于确定是否要从计算中消除 `expr` 的NULL值。默认情况下包括NULL值，这意味着如果在过滤结果中的第一个值为NULL，则返回的是NULL。如果指定了IGNORE NULLS，则返回过滤结果中第一个非NULL值。如果所有值都是NULL，即使指定了IGNORE NULLS，仍将返回NULL。

示例：

假设我们有以下数据：

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

使用FIRST_VALUE()基于国家分组，返回每个分组中第一个greeting值。

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

返回当前行的值向后滞后`offset`行的行的值。此函数通常用于在行之间比较值和过滤数据。

`LAG()`可用于查询以下类型的数据：

* 数值: TINYINT, SMALLINT, INT, BIGINT, LARGEINT, FLOAT, DOUBLE, DECIMAL
* 字符串: CHAR, VARCHAR
* 日期: DATE, DATETIME
* 从StarRocks v2.5开始支持BITMAP和HLL。

语法：

```SQL
LAG(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

参数：

* `expr`：您要计算的字段。
* `offset`：偏移。它必须是**正整数**。如果未指定此参数，则默认为1。
* `default`：如果未找到匹配行，则返回的默认值。如果未指定此参数，则默认为NULL。`default`支持与`expr`兼容的任何表达式。
* `IGNORE NULLS` 是从 v3.0 开始支持的。它用于确定是否在结果中包括 `expr` 的 NULL 值。默认情况下，在计算 `offset` 行时会包括 NULL 值，这意味着如果目标行的值为 NULL，则返回 NULL。请参见示例 1。如果指定了 IGNORE NULLS，则在计算 `offset` 行时会忽略 NULL 值，系统会继续搜索 `offset` 个非空值。如果找不到 `offset` 个非空值，则返回 NULL 或 `default`（如果指定了）。请参见示例 2。

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

查询此表中的数据，其中 `offset` 为 2，这意味着遍历前两行；`default` 为 0，这意味着如果找不到匹配的行，则返回 0。

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

对于前两行，不存在前两行，因此返回默认值 0。

对于第 3 行中的 NULL，向前推两行的值也为 NULL，并且会返回 NULL，因为允许 NULL 值。

示例 2：指定了 IGNORE NULLS

使用前面的表和参数设置。

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

对于第 1 到 4 行，系统无法在前面的行中找到两个非 NULL 值，因此返回默认值 0。

对于第 7 行中的值 6，向前推两行的值为 NULL，并且由于指定了 IGNORE NULLS，忽略了 NULL 值。系统继续搜索非空值，并返回第 4 行中的值 2。
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


对于第7至10行，系统找不到后续行中的两个非空值，因此返回默认值0。

对于第一行，向前两行的值为NULL，因为指定了IGNORE NULLS，NULL被忽略。系统继续寻找第二个非空值，并返回第4行的2。

### MAX()

返回当前窗口中指定行的最大值。

语法

```SQL
MAX(expr) [OVER (analytic_clause)]
```

例子:

计算从当前行到下一行的最大值。

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

从StarRocks 2.4开始，您可以将行范围指定为`rows between n preceding and n following`，这意味着您可以捕获当前行之前的`n`行和当前行之后的`n`行。

例子语句:

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

语法:

```SQL
MIN(expr) [OVER (analytic_clause)]
```

例子:

计算从当前行到下一行的最小值。

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

从StarRocks 2.4开始，您可以将行范围指定为`rows between n preceding and n following`，这意味着您可以捕获当前行之前的`n`行和当前行之后的`n`行。

例子语句:

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

PERCENT_RANK()函数计算结果集中行的相对排名作为百分比。它返回除最高值外当前行值以下分区值的百分比。返回值的范围为0到1。此函数对百分位数计算和数据分布分析很有用。

PERCENT_RANK()函数使用以下公式计算，其中rank表示行排名，rows表示分区行数:

```plaintext
(rank - 1) / (rows - 1)
```

语法:

```SQL
PERCENT_RANK() OVER (partition_by_clause order_by_clause)
```

**应该将此函数与ORDER BY一起使用，以便将分区行排序为所需顺序。如果没有ORDER BY，所有行将被视为同列，并且具有值(1 - 1)/(N - 1) = 0，其中N是分区大小。**

以下示例显示了列y在列x的每个分组中的相对排名。

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

RANK()函数用于表示排名。与DENSE_RANK()不同，RANK()将**显示为空位**的数字。例如，如果出现两个并列的1，RANK()的第三个数字将是3，而不是2。

语法:

```SQL
RANK() OVER(partition_by_clause order_by_clause)
```

例子:

根据列x进行排名:

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

返回每个分区的行的连续递增整数，起始值为1。与RANK()和DENSE_RANK()不同，ROW_NUMBER()返回的值**不会重复或有间隙**，并且**连续递增**。

语法:

```SQL
ROW_NUMBER() OVER(partition_by_clause order_by_clause)
```

例子:

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

QUALIFY子句用于过滤窗口函数的结果。在SELECT语句中，您可以使用QUALIFY子句将条件应用于要过滤的列以过滤结果。QUALIFY类似于聚合函数中的HAVING子句。从v2.5开始支持此函数。

QUALIFY简化了SELECT语句的编写。

在使用QUALIFY之前，SELECT语句可能如下所示:

```SQL
SELECT *
FROM (SELECT DATE,
             PROVINCE_CODE,
             TOTAL_SCORE,
             ROW_NUMBER() OVER(PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) AS SCORE_ROWNUMBER
      FROM example_table) T1
WHERE T1.SCORE_ROWNUMBER = 1;
```

使用QUALIFY后，语句缩短为:

```SQL
SELECT DATE, PROVINCE_CODE, TOTAL_SCORE
FROM example_table 
QUALIFY ROW_NUMBER() OVER(PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) = 1;
```

QUALIFY仅支持以下三个窗口函数: ROW_NUMBER(), RANK()和DENSE_RANK()。

**语法:**

```SQL
SELECT <column_list>
FROM <data_source>
[GROUP BY ...]
[HAVING ...]
QUALIFY <window_function>
[ ... ]
```

**参数:**



`<column_list>`: 欲获得数据的列。

`<data_source>`: 数据源通常是一个表。

`<window_function>`: “QUALIFY”子句后只能跟着窗口函数，包括ROW_NUMBER()、RANK()和DENSE_RANK()等。

**示例:**

```SQL
-- 创建一个表。
CREATE TABLE sales_record (
   city_id INT,
   item STRING,

   sales INT
) DISTRIBUTED BY HASH(`city_id`);

-- 向表中插入数据。
insert into sales_record values
(1,'fruit',95),

(2,'drinks',70),
(3,'fruit',87),
(4,'drinks',98);

-- 从表中查询数据。
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

示例1: 获取表中行号大于1的记录。

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

示例2: 获取表中每个分区的行号为1的记录。表通过`item`分成两个分区，并返回每个分区中的第一行。

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

示例3: 获取表中每个分区的销售排名第1的记录。表通过`item`分成两个分区，并返回每个分区中销售额最高的记录。

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


**使用说明:**

带有QUALIFY的查询中，子句的执行顺序如下：

> 1. From
> 2. Where
> 3. Group by

> 4. Having

> 5. Window

> 6. QUALIFY
> 7. Distinct
> 8. Order by
> 9. Limit

### SUM()

语法:

```SQL

SUM(expr) [OVER (analytic_clause)]
```

示例:

按属性分组，并计算组内**当前行、前一行和后一行**的总和。

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

### VARIANCE, VAR_POP, VARIANCE_POP

返回表达式的总体方差。VAR_POP和VARIANCE_POP是VAR的别名。这些函数可以作为窗口函数使用，自v2.5.10起支持。

**语法:**

```SQL

VARIANCE(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])

```

> **注意**
>
> 从2.5.13、3.0.7、3.1.4版本开始，此窗口函数支持ORDER BY和Window子句。

**参数:**

如果`expr`是表列，必须评估为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

**示例:**

假设表`agg`有如下数据:

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


使用VARIANCE()函数。

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

### VAR_SAMP, VARIANCE_SAMP

返回表达式的样本方差。这些函数可以作为窗口函数使用，自v2.5.10起支持。

**语法:**

```sql
VAR_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 从2.5.13、3.0.7、3.1.4版本开始，此窗口函数支持ORDER BY和Window子句。

**参数:**

如果`expr`是表列，必须评估为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

**示例:**

假设表`agg`有如下数据:

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
从2.5.13、3.0.7、3.1.4版本开始，此窗口函数支持ORDER BY和WINDOW子句。

**参数：**

如果`expr`是表列，则其必须求值为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

**示例：**

假设表`agg`具有以下数据：

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

使用STD()窗口函数。

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

返回表达式的样本标准差。此函数自版本v2.5.10起可用作窗口函数。

**语法：**

```sql
STDDEV_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 从2.5.13、3.0.7、3.1.4版本开始，此窗口函数支持ORDER BY和WINDOW子句。

**参数：**

如果`expr`是表列，则其必须求值为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

**示例：**

假设表`agg`具有以下数据：

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

使用STDDEV_SAMP()窗口函数。

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

返回两个表达式的样本协方差。此函数从版本v2.5.10起受支持。它也是一个聚合函数。

**语法：**

```sql
COVAR_SAMP(expr1,expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 从2.5.13、3.0.7、3.1.4版本开始，此窗口函数支持ORDER BY和WINDOW子句。

**参数：**

如果`expr`是表列，则其必须求值为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

**示例：**

假设表`agg`具有以下数据：

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

使用COVAR_SAMP()窗口函数。

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

返回两个表达式的总体协方差。此函数从版本v2.5.10起受支持。它也是一个聚合函数。

**语法：**

```sql
COVAR_POP(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 从2.5.13、3.0.7、3.1.4版本开始，此窗口函数支持ORDER BY和WINDOW子句。

**参数：**

如果`expr`是表列，则其必须求值为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

**示例：**

假设表`agg`具有以下数据：

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

使用COVAR_POP()窗口函数。

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

返回两个表达式之间的Pearson相关系数。此函数自版本v2.5.10起受支持。它也是一个聚合函数。

**语法：**

```sql
CORR(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 从2.5.13、3.0.7、3.1.4版本开始，此窗口函数支持ORDER BY和WINDOW子句。

**参数：**

如果`expr`是表列，则其必须求值为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

```plaintext

假设表“agg”有以下数据：

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

使用CORR()窗口函数。

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