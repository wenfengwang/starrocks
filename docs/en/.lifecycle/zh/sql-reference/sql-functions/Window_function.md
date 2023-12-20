---
displayed_sidebar: English
---


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

如上例所示，当`num_buckets`为2时：

* 编号1至6的行被划分到第一个分区；编号1至3的行存放在第一个桶中，编号4至6的行存放在第二个桶中。
* 编号7至9的行被划分到第二个分区；编号7和8的行存储在第一个桶中，编号9的行存储在第二个桶中。
* 编号10的行被划分到第三个分区并存储在第一个桶中。

<br/>


### FIRST_VALUE()

`FIRST_VALUE()` 返回窗口范围内的**第一个**值。

语法：

```SQL
FIRST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

从 v2.5.0 开始支持 `IGNORE NULLS`。用于判断 `expr` 的 NULL 值是否从计算中剔除。默认情况下，包含 NULL 值，这意味着如果过滤结果中的第一个值为 NULL，则返回 NULL。如果指定 `IGNORE NULLS`，则返回筛选结果中的第一个非空值。如果所有值均为 NULL，则即使指定 `IGNORE NULLS`，也会返回 NULL。

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

使用 `FIRST_VALUE()` 根据国家分组返回每个分组中的第一个问候语值。

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

`LAG()` 返回落后当前行 `offset` 行的行的值。该函数通常用于比较行之间的值和过滤数据。

`LAG()` 可用于查询以下类型的数据：

* 数字：TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL
* 字符串：CHAR、VARCHAR
* 日期：DATE、DATETIME
* 从 StarRocks v2.5 开始支持 BITMAP 和 HLL。

语法：

```SQL
LAG(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

参数：

* `expr`：您要计算的字段。
* `offset`：偏移量。它必须是一个**正整数**。如果不指定该参数，则默认为 1。
* `default`：如果没有找到匹配的行，则返回默认值。如果不指定该参数，则默认为 NULL。`default` 支持任何类型与 `expr` 兼容的表达式。
* 从 v3.0 开始支持 `IGNORE NULLS`。用于判断结果中是否包含 `expr` 的 NULL 值。默认情况下，统计偏移行时会包含 NULL 值，即如果目标行值为 NULL，则返回 NULL。请参见示例 1。如果指定 `IGNORE NULLS`，则在对偏移行进行计数时忽略 NULL 值，并且系统继续搜索偏移非空值。如果找不到偏移量非空值，则返回 NULL 或默认值（如果指定）。参见示例 2。

示例 1：未指定 `IGNORE NULLS`

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

从此表中查询数据，其中 `offset` 为 2，表示遍历前两行；`default` 为 0，这意味着如果没有找到匹配的行，则返回 0。

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

对于前两行，不存在前两行，并且返回默认值 0。

对于第 3 行中的 NULL，向后两行的值是 NULL，并且返回 NULL，因为允许 NULL 值。

示例 2：指定 `IGNORE NULLS`

使用上表和参数设置。

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
返回表达式的总体方差。VAR_POP 和 VARIANCE_POP 是 VARIANCE 的别名。自 v2.5.10 起，这些函数可以作为窗口函数使用。

**句法：**

```SQL
VARIANCE(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，此窗口函数支持 ORDER BY 和 Window 子句。

**参数：**

如果 `expr` 是表列，它必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例：**

假设表 `agg` 有以下数据：

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

### VAR_SAMP, VARIANCE_SAMP

返回表达式的样本方差。自 v2.5.10 起，这些函数可以作为窗口函数使用。

**句法：**

```sql
VAR_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，此窗口函数支持 ORDER BY 和 Window 子句。

**参数：**

如果 `expr` 是表列，它必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例：**

假设表 `agg` 有以下数据：

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

### STD, STDDEV, STDDEV_POP

返回表达式的标准差。自 v2.5.10 起，这些函数可以作为窗口函数使用。

**句法：**

```sql
STD(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，此窗口函数支持 ORDER BY 和 Window 子句。

**参数：**

如果 `expr` 是表列，它必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例：**

假设表 `agg` 有以下数据：

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

返回表达式的样本标准差。自 v2.5.10 起，此函数可以作为窗口函数使用。

**句法：**

```sql
STDDEV_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，此窗口函数支持 ORDER BY 和 Window 子句。

**参数：**

如果 `expr` 是表列，它必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例：**

假设表 `agg` 有以下数据：

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

返回两个表达式的样本协方差。自 v2.5.10 起支持此函数。它也是一个聚合函数。

**句法：**

```sql
COVAR_SAMP(expr1,expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，此窗口函数支持 ORDER BY 和 Window 子句。

**参数：**

如果 `expr` 是表列，它必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例：**

假设表 `agg` 有以下数据：

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

返回两个表达式的总体协方差。自 v2.5.10 起支持此函数。它也是一个聚合函数。

**句法：**

```sql
COVAR_POP(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，此窗口函数支持 ORDER BY 和 Window 子句。

**参数：**

如果 `expr` 是表列，它必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例：**

假设表 `agg` 有以下数据：

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

返回两个表达式之间的皮尔逊相关系数。自 v2.5.10 起支持此函数。它也是一个聚合函数。

**句法：**

```sql
CORR(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注意**
> 从 2.5.13、3.0.7、3.1.4 版本开始，此窗口函数支持 ORDER BY 和 Window 子句。

**参数：**

如果 `expr` 是表列，它必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

**示例：**

假设表 `agg` 有以下数据：

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
```