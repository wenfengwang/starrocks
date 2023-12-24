---
displayed_sidebar: English
---

# 查询

## 描述

从一个或多个表、视图或物化视图中查询数据。SELECT 语句通常由以下子句组成：

- [WITH](#with)
- [WHERE 和运算符](#where-and-operators)
- [GROUP BY](#group-by)
- [HAVING](#having)
- [UNION](#union)
- [INTERSECT](#intersect)
- [EXCEPT/MINUS](#exceptminus)
- [ORDER BY](#order-by)
- [LIMIT](#limit)
- [OFFSET](#offset)
- [JOIN](#join)
- [子查询](#subquery)
- [DISTINCT](#distinct)
- [别名](#alias)

SELECT 可以作为独立语句，也可以作为嵌套在其他语句中的子句。SELECT 子句的输出可以作为其他语句的输入。

StarRocks 的查询语句基本符合 SQL92 标准。以下是支持的 SELECT 用法的简要说明。

> **注意**
>
> 要查询 StarRocks 内部表、视图或物化视图中的数据，您必须拥有这些对象的 SELECT 权限。要查询外部数据源中的表、视图或物化视图的数据，您必须对相应的外部目录具有 USAGE 权限。

### WITH

可以在 SELECT 语句之前添加的子句，用于定义在 SELECT 中多次引用的复杂表达式的别名。

与 CREATE VIEW 类似，但子句中定义的表名和列名在查询结束后不会保留，并且不会与实际表或 VIEW 中的名称冲突。

使用 WITH 子句的好处是：

方便且易于维护，减少查询中的重复。

通过将查询中最复杂的部分抽象为单独的块，可以更轻松地阅读和理解 SQL 代码。

例子：

```sql
-- 在 UNION ALL 查询的初始阶段，在外部级别定义一个子查询，在内部级别定义另一个子查询。

with t1 as (select 1), t2 as (select 2)
select * from t1 union all select * from t2;
```

### JOIN

联接操作将两个或多个表中的数据组合在一起，然后从其中一些表中返回某些列的结果集。

StarRocks 支持自连接、交叉连接、内连接、外连接、半连接和反连接。外连接包括左连接、右连接和完全连接。

语法：

```sql
SELECT select_list FROM
table_or_subquery1 [INNER] JOIN table_or_subquery2 |
table_or_subquery1 {LEFT [OUTER] | RIGHT [OUTER] | FULL [OUTER]} JOIN table_or_subquery2 |
table_or_subquery1 {LEFT | RIGHT} SEMI JOIN table_or_subquery2 |
table_or_subquery1 {LEFT | RIGHT} ANTI JOIN table_or_subquery2 |
[ ON col1 = col2 [AND col3 = col4 ...] |
USING (col1 [, col2 ...]) ]
[other_join_clause ...]
[ WHERE where_clauses ]
```

```sql
SELECT select_list FROM
table_or_subquery1, table_or_subquery2 [, table_or_subquery3 ...]
[other_join_clause ...]
WHERE
col1 = col2 [AND col3 = col4 ...]
```

```sql
SELECT select_list FROM
table_or_subquery1 CROSS JOIN table_or_subquery2
[other_join_clause ...]
[ WHERE where_clauses ]
```

#### 自连接

StarRocks 支持自连接，即自连接和自连接。例如，联接同一表的不同列。

实际上没有特殊的语法来标识自连接。自连接中联接的两端条件来自同一个表。

我们需要为它们分配不同的别名。

例子：

```sql
SELECT lhs.id, rhs.parent, lhs.c1, rhs.c2 FROM tree_data lhs, tree_data rhs WHERE lhs.id = rhs.parent;
```

#### 交叉连接

交叉连接会产生很多结果，因此交叉连接应谨慎使用。

即使需要使用交叉联接，也需要使用筛选条件，并确保返回的结果更少。例：

```sql
SELECT * FROM t1, t2;

SELECT * FROM t1 CROSS JOIN t2;
```

#### 内连接

内连接是最著名和最常用的连接。返回两个相似表请求的列的结果，如果两个表的列包含相同的值，则联接。

如果两个表的列名相同，我们需要使用全名（table_name.column_name 的形式）或列名别名。

例子：

以下三个查询是等效的。

```sql
SELECT t1.id, c1, c2 FROM t1, t2 WHERE t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```

#### 外连接

外连接返回左表或右表或两者的所有行。如果另一个表中没有匹配的数据，请将其设置为 NULL。例：

```sql
SELECT * FROM t1 LEFT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 RIGHT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;
```

#### 等价连接和不等连接

通常，用户使用最相等的连接，这要求连接条件的运算符为等号。

不等连接可用于联接条件！=，等号。不相等的连接会产生大量结果，并且可能会在计算过程中超出内存限制。

请谨慎使用。不等连接仅支持内部连接。例：

```sql
SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id > t2.id;
```

#### 半连接

左半连接仅返回左表中与右表中的数据匹配的行，而不考虑与右表中的数据匹配的行数。

左表的这一行最多返回一次。右半连接的工作方式类似，只不过返回的数据是右表。

例子：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT SEMI JOIN t2 ON t1.id = t2.id;
```

#### 反连接

左反连接仅返回左表中与右表不匹配的行。

右反连接反转此比较，仅返回右表中与左表不匹配的行。例：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT ANTI JOIN t2 ON t1.id = t2.id;
```

#### 等连接和非等连接

StarRocks 支持的各种 join 可以分为 equi-joins 和非 equi-join，具体取决于 join 中指定的 join 条件。

| **Equi****-joins** | 自连接、交叉连接、内连接、外连接、半连接和反连接 |
| -------------------------- | ------------------------------------------------------------ |
| **非****等****联接** | 交叉连接、内连接、左半连接、左反连接和外连接   |

- 等连接
  
  等连接使用连接条件，其中两个连接项由 `=` 运算符组合。示例： `a JOIN b ON a.id = b.id`.

- 非等连接
  
  非等连接使用连接条件，在该条件中，两个连接项由比较运算符（如 `<`、 、 `<=`, `>` `>=`或 `<>` ）组合在一起。示例：`a JOIN b ON a.id < b.id`.非等式连接的运行速度比等式连接慢。我们建议您在使用非等连接时谨慎行事。

  以下两个示例演示如何运行非 equi-joins：

  ```SQL
  SELECT t1.id, c1, c2 
  FROM t1 
  INNER JOIN t2 ON t1.id < t2.id;
    
  SELECT t1.id, c1, c2 
  FROM t1 
  LEFT JOIN t2 ON t1.id > t2.id;
  ```

### ORDER BY

SELECT 语句的 ORDER BY 子句通过比较一列或多列中的值对结果集进行排序。

ORDER BY 是一项耗时且耗费资源的操作，因为必须先将所有结果发送到一个节点进行合并，然后才能对结果进行排序。排序比没有 ORDER BY 的查询消耗更多的内存资源。

因此，如果只需要排序结果集中的前 `N` 个结果，则可以使用 LIMIT 子句，这样可以减少内存使用量和网络开销。如果未指定 LIMIT 子句，则缺省情况下返回前 65535 个结果。

语法：

```sql
ORDER BY <column_name> 
    [ASC | DESC]
    [NULLS FIRST | NULLS LAST]
```

`ASC` 指定应按升序返回结果。 `DESC` 指定应按降序返回结果。如果未指定顺序，则 ASC（升序）是默认值。例：

```sql
select * from big_table order by tiny_column, short_column desc;
```

NULL 值的排序顺序： `NULLS FIRST` 指示应在非 NULL 值之前返回 NULL 值。 `NULLS LAST` 指示应在非 NULL 值之后返回 NULL 值。

例子：

```sql
select  *  from  sales_record  order by  employee_id  nulls first;
```

### GROUP BY

GROUP BY 子句通常与聚合函数一起使用，例如 COUNT（）、SUM（）、AVG（）、MIN（） 和 MAX（）。

GROUP BY 指定的列将不参与聚合操作。可以将 GROUP BY 子句与 HAVING 子句一起添加，以筛选聚合函数生成的结果。

例子：

```sql
select tiny_column, sum(short_column)
from small_table 
group by tiny_column;
```

```plain text
+-------------+---------------------+
| tiny_column |  sum('short_column')|
+-------------+---------------------+
|      1      |        2            |
|      2      |        1            |
+-------------+---------------------+
```

#### 语法

  ```sql
  SELECT ...
  FROM ...
  [ ... ]
  GROUP BY [
      , ... |
      GROUPING SETS [, ...] (  groupSet [ , groupSet [ , ... ] ] ) |
      ROLLUP(expr  [ , expr [ , ... ] ]) |
      expr  [ , expr [ , ... ] ] WITH ROLLUP |
      CUBE(expr  [ , expr [ , ... ] ]) |
      expr  [ , expr [ , ... ] ] WITH CUBE
      ]
  [ ... ]
  ```

#### 参数

  `groupSet` 表示由“选择”列表中的列、别名或表达式组成的集合。  `groupSet ::= { ( expr  [ , expr [ , ... ] ] )}`

  `expr`  指示选择列表中的列、别名或表达式。

#### 注意

StarRocks 支持 PostgreSQL 等语法。语法示例如下：

  ```sql
  SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
  SELECT a, b,c, SUM( d ) FROM tab1 GROUP BY ROLLUP(a,b,c)
  SELECT a, b,c, SUM( d ) FROM tab1 GROUP BY CUBE(a,b,c)
  ```

  `ROLLUP(a,b,c)` 等同于以下`GROUPING SETS` 语句：

  ```sql
  GROUPING SETS (
  (a,b,c),
  ( a, b ),
  ( a),
  ( )
  )
  ```

  `CUBE ( a, b, c )`  等同于以下`GROUPING SETS` 语句：

  ```sql
  GROUPING SETS (
  ( a, b, c ),
  ( a, b ),
  ( a,    c ),
  ( a       ),
  (    b, c ),

  (    b    ),
  (       c ),
  (         )
  )
  ```

#### 例子

  以下是实际数据的示例：

  ```plain text
  SELECT * FROM t;
  +------+------+------+
  | k1   | k2   | k3   |
  +------+------+------+
  | a    | A    |    1 |
  | a    | A    |    2 |
  | a    | B    |    1 |
  | a    | B    |    3 |
  | b    | A    |    1 |
  | b    | A    |    4 |
  | b    | B    |    1 |
  | b    | B    |    5 |
  +------+------+------+
  8 行在集合中 (0.01 秒)

  SELECT k1, k2, SUM(k3) FROM t GROUP BY GROUPING SETS ( (k1, k2), (k2), (k1), ( ) );
  +------+------+-----------+
  | k1   | k2   | sum(`k3`) |
  +------+------+-----------+
  | b    | B    |         6 |
  | a    | B    |         4 |
  | a    | A    |         3 |
  | b    | A    |         5 |
  | NULL | B    |        10 |
  | NULL | A    |         8 |
  | a    | NULL |         7 |
  | b    | NULL |        11 |
  | NULL | NULL |        18 |
  +------+------+-----------+
  9 行在集合中 (0.06 秒)

  > SELECT k1, k2, GROUPING_ID(k1,k2), SUM(k3) FROM t GROUP BY GROUPING SETS ((k1, k2), (k1), (k2), ());
  +------+------+---------------+----------------+
  | k1   | k2   | grouping_id(k1,k2) | sum(`k3`) |
  +------+------+---------------+----------------+
  | a    | A    |             0 |              3 |
  | a    | B    |             0 |              4 |
  | a    | NULL |             1 |              7 |
  | b    | A    |             0 |              5 |
  | b    | B    |             0 |              6 |
  | b    | NULL |             1 |             11 |
  | NULL | A    |             2 |              8 |
  | NULL | B    |             2 |             10 |
  | NULL | NULL |             3 |             18 |
  +------+------+---------------+----------------+
  9 行在集合中 (0.02 秒)
  ```

GROUP BY `GROUPING SETS` ｜ `CUBE` ｜ `ROLLUP` 是 GROUP BY 子句的扩展。它可以实现在 GROUP BY 子句中对多个集合的组进行聚合。结果等效于多个对应的 GROUP BY 子句的 UNION 运算。

GROUP BY 子句是 GROUP BY GROUPING SETS 仅包含一个元素的特例。例如，GROUPING SETS 语句：

  ```sql
  SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
  ```

  查询结果等效于：

  ```sql
  SELECT a, b, SUM( c ) FROM tab1 GROUP BY a, b
  UNION
  SELECT a, null, SUM( c ) FROM tab1 GROUP BY a
  UNION
  SELECT null, b, SUM( c ) FROM tab1 GROUP BY b
  UNION
  SELECT null, null, SUM( c ) FROM tab1
  ```

  `GROUPING(expr)` 指示列是否为聚合列。如果是聚合列，则为 0，否则为 1。

  `GROUPING_ID(expr  [ , expr [ , ... ] ])` 类似于 GROUPING。GROUPING_ID 根据指定的列顺序计算列列表的位图值，每个位都是 GROUPING 的值。
  
  GROUPING_ID() 函数返回位向量的十进制值。

### HAVING

HAVING 子句不过滤表中的行数据，而是过滤聚合函数的结果。

一般来说，HAVING 与聚合函数（如 COUNT()、SUM()、AVG()、MIN()、MAX()）和 GROUP BY 子句一起使用。

例子：

```sql
select tiny_column, sum(short_column) 
from small_table 
group by tiny_column 
having sum(short_column) = 1;
```

```plain text
+-------------+---------------------+
|tiny_column  | sum('short_column') |
+-------------+---------------------+
|         2   |        1            |
+-------------+---------------------+

1 行在集合中 (0.07 秒)
```

```sql
select tiny_column, sum(short_column) 
from small_table 
group by tiny_column 
having tiny_column > 1;
```

```plain text
+-------------+---------------------+
|tiny_column  | sum('short_column') |
+-------------+---------------------+
|      2      |          1          |
+-------------+---------------------+

1 行在集合中 (0.07 秒)
```

### LIMIT

LIMIT 子句用于限制返回的最大行数。设置返回的最大行数可以帮助 StarRocks 优化内存使用。

该子句主要用于以下场景：

返回前 N 个查询的结果。

考虑下表中包含的内容。

由于表中的数据量较大，或者由于 where 子句没有过滤太多数据，因此需要限制查询结果集的大小。

使用说明：LIMIT 子句的值必须是数字文字常量。

例子：

```plain text
mysql> select tiny_column from small_table limit 1;

+-------------+
|tiny_column  |
+-------------+
|     1       |
+-------------+

1 行在集合中 (0.02 秒)
```

```plain text
mysql> select tiny_column from small_table limit 10000;

+-------------+
|tiny_column  |
+-------------+
|      1      |
|      2      |
+-------------+

2 行在集合中 (0.01 秒)
```

### OFFSET

OFFSET 子句使结果集跳过前几行，并直接返回以下结果。

结果集默认从第 0 行开始，因此偏移量 0 和无偏移量返回相同的结果。

一般来说，OFFSET 子句需要与 ORDER BY 和 LIMIT 子句一起使用才能有效。

例子：

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 3;

+----------------+
| varchar_column | 
+----------------+
|    beijing     | 
|    chongqing   | 
|    tianjin     | 
+----------------+

3 行在集合中 (0.02 秒)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 0;

+----------------+
|varchar_column  |
+----------------+
|     beijing    |
+----------------+

1 行在集合中 (0.01 秒)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 1;

+----------------+
|varchar_column  |
+----------------+
|    chongqing   | 
+----------------+

1 行在集合中 (0.01 秒)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 2;

+----------------+
|varchar_column  |
+----------------+
|     tianjin    |     
+----------------+

1 行在集合中 (0.02 秒)
```

注意：允许使用不带 order by 的 offset 语法，但此时 offset 没有意义。

在这种情况下，仅取极限值，而忽略偏移值。所以没有命令。

偏移量超过结果集中的最大行数，并且仍然是结果。建议用户将偏移量与顺序依据一起使用。

### UNION

合并多个查询的结果。

**语法：**

```sql
query_1 UNION [DISTINCT | ALL] query_2
```

- DISTINCT（默认值）仅返回唯一行。UNION 等同于 UNION DISTINCT。
- ALL 合并所有行，包括重复行。由于重复数据消除会占用大量内存，因此使用 UNION ALL 的查询速度更快，占用的内存更少。为了获得更好的性能，请使用 UNION ALL。

> **注意**
>
> 每个查询语句必须返回相同数量的列，并且这些列必须具有兼容的数据类型。

**例子：**

创建表 `select1` 和 `select2`.

```SQL
CREATE TABLE select1(
    id          INT,
    price       INT
    )
DISTRIBUTED BY HASH(id);

INSERT INTO select1 VALUES
    (1,2),
    (1,2),
    (2,3),
    (5,6),
    (5,6);

CREATE TABLE select2(
    id          INT,
    price       INT
    )
DISTRIBUTED BY HASH(id);

INSERT INTO select2 VALUES
    (2,3),
    (3,4),
    (5,6),
    (7,8);
```

示例 1：返回两个表中的所有 ID，包括重复项。

```Plaintext
mysql> (select id from select1) union all (select id from select2) order by id;

+------+
| id   |
+------+
|    1 |
|    1 |
|    2 |
|    2 |
|    3 |
|    5 |
|    5 |
|    5 |
|    7 |
+------+
11 行在集合中 (0.02 秒)
```

示例 2：返回两个表中的所有唯一 ID。以下两个语句是等效的。

```Plaintext
mysql> (select id from select1) union (select id from select2) order by id;

+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
|    5 |
|    7 |
+------+
6 行在集合中 (0.01 秒)

mysql> (select id from select1) union distinct (select id from select2) order by id;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
|    5 |
|    7 |
+------+
5 行在集合中 (0.02 秒)
```

示例3：返回两个表中所有唯一ID中的前三个ID。以下两个语句是等效的。

```SQL
mysql> (select id from select1) union distinct (select id from select2)
order by id
limit 3;
++------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
4 行在集合中 (0.11 秒)

mysql> select * from (select id from select1 union distinct select id from select2) as t1
order by id
limit 3;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 行在集合中 (0.01 秒)
```

### **相交**

计算多个查询结果的交集，即所有结果集中显示的结果。此子句仅返回结果集中的唯一行。不支持 ALL 关键字。

**语法:**

```SQL
query_1 INTERSECT [DISTINCT] query_2
```

> **注意**
>
> - INTERSECT 等同于 INTERSECT DISTINCT。
> - 每个查询语句必须返回相同数量的列，并且这些列必须具有兼容的数据类型。

**例子:**

使用 UNION 中的两个表。

返回 `(id, price)` 两个表共有的不同组合。以下两个语句是等效的。

```Plaintext
mysql> (select id, price from select1) intersect (select id, price from select2)
order by id;

+------+-------+
| id   | price |
+------+-------+
|    2 |     3 |
|    5 |     6 |
+------+-------+

mysql> (select id, price from select1) intersect distinct (select id, price from select2)
order by id;

+------+-------+
| id   | price |
+------+-------+
|    2 |     3 |
|    5 |     6 |
+------+-------+
```

### **除/减号**

返回左侧查询中不存在的不同结果。EXCEPT 等同于 MINUS。

**语法:**

```SQL
query_1 {EXCEPT | MINUS} [DISTINCT] query_2
```

> **注意**
>
> - EXCEPT 等同于 EXCEPT DISTINCT。不支持 ALL 关键字。
> - 每个查询语句必须返回相同数量的列，并且这些列必须具有兼容的数据类型。


**示例：**

使用 UNION 中的两个表。

返回 `select1` 中无法在 `select2` 中找到的不同的 `(id, price)` 组合。

```Plaintext
mysql> (select id, price from select1) except (select id, price from select2)
order by id;
+------+-------+
| id   | price |
+------+-------+
|    1 |     2 |
+------+-------+

mysql> (select id, price from select1) except (select id, price from select2)
order by id;
+------+-------+
| id   | price |
+------+-------+
|    1 |     2 |
+------+-------+
```

### DISTINCT

DISTINCT 关键字用于对结果集进行去重。例如：

```SQL
-- 返回一列的唯一值。
select distinct tiny_column from big_table limit 2;

-- 返回多列值的唯一组合。
select distinct tiny_column, int_column from big_table limit 2;
```

DISTINCT 可以与聚合函数（通常是计数函数）一起使用，而 count (distinct) 用于计算一列或多列上包含多少个不同的组合。

```SQL
-- 计算一列的唯一值。
select count(distinct tiny_column) from small_table;
```

```plain text
+-------------------------------+
| count(DISTINCT 'tiny_column') |
+-------------------------------+
|             2                 |
+-------------------------------+
1 row in set (0.06 sec)
```

```SQL
-- 计算多列值的唯一组合。
select count(distinct tiny_column, int_column) from big_table limit 2;
```

StarRocks 支持同时使用 DISTINCT 的多个聚合函数。

```SQL
-- 分别计算多个聚合函数的唯一值。
select count(distinct tiny_column, int_column), count(distinct varchar_column) from big_table;
```

### 子查询

子查询根据相关性分为两种类型：

- 不相关子查询独立于其外部查询获取其结果。
- 相关子查询需要其外部查询中的值。

#### 不相关子查询

不相关子查询支持 [NOT] IN 和 EXISTS。

例：

```sql
SELECT x FROM t1 WHERE x [NOT] IN (SELECT y FROM t2);

SELECT * FROM t1 WHERE (x,y) [NOT] IN (SELECT x,y FROM t2 LIMIT 2);

SELECT x FROM t1 WHERE EXISTS (SELECT y FROM t2 WHERE y = 1);
```

从 v3.0 开始，您可以在 `SELECT... FROM... WHERE... [NOT] IN` 的 WHERE 子句中指定多个字段，例如，在第二个 SELECT 语句中使用 `WHERE (x,y)`。

#### 相关子查询

相关子查询支持 [NOT] IN 和 [NOT] EXISTS。

例：

```sql
SELECT * FROM t1 WHERE x [NOT] IN (SELECT a FROM t2 WHERE t1.y = t2.b);

SELECT * FROM t1 WHERE [NOT] EXISTS (SELECT a FROM t2 WHERE t1.y = t2.b);
```

子查询还支持标量量子查询。它可以分为无关标量量子查询、相关标量量子查询和作为一般函数参数的标量量子查询。

例子：

1. 使用带有谓词 = 的无关标量量子查询。例如，输出有关工资最高的人的信息。

    ```sql
    SELECT name FROM table WHERE salary = (SELECT MAX(salary) FROM table);
    ```

2. 使用带有 `>`、`<` 等谓词的无关标量量子查询。例如，输出有关薪水高于平均水平的人的信息。

    ```sql
    SELECT name FROM table WHERE salary > (SELECT AVG(salary) FROM table);
    ```

3. 相关标量量子查询。例如，输出每个部门的最高工资信息。

    ```sql
    SELECT name FROM table a WHERE salary = (SELECT MAX(salary) FROM table b WHERE b.Department= a.Department);
    ```

4. 将标量量子查询用作普通函数的参数。

    ```sql
    SELECT name FROM table WHERE salary = abs((SELECT MAX(salary) FROM table));
    ```

### WHERE 和运算符

SQL 运算符是一系列用于比较的函数，广泛用于 select 语句的 where 子句中。

#### 算术运算符

算术运算符通常出现在包含左、右和最常见的左操作数的表达式中。

**+ 和 -**：可以作为一个单位使用，也可以作为二元运算符使用。作为单位运算符时，例如 +1、-2.5 或 -col_ name，意味着该值乘以 +1 或 -1。

因此，单元运算符 + 返回一个不变的值，而单元运算符 - 更改该值的符号位。

用户可以叠加两个单元运算符，例如 +5（返回正值）、-+2 或 +2（返回负值），但用户不能使用两个连续的 - 符号。

因为--在以下语句中被解释为注释（当用户可以使用两个符号时，两个符号之间需要空格或括号，例如 -（-2） 或 - -2，这实际上会导致 + 2）。

当 + 或 - 是二元运算符时，例如 2+2、3+1.5 或 col1+col2，则表示从右侧值中添加或减去左侧值。左值和右值都必须是数值类型。

**and 和 /**：分别表示乘法和除法。两端的操作数必须是数据类型。当两个数字相乘时。

如果需要，可以提升较小的操作数（例如，将 SMALLINT 提升为 INT 或 BIGINT），并且表达式的结果将被提升到下一个较大的类型。

例如，TINYINT 乘以 INT 将产生 BIGINT 类型的结果。当两个数字相乘时，操作数和表达式结果都被解释为 DOUBLE 类型，以避免精度损失。

如果用户想要将表达式的结果转换为另一种类型，则需要使用 CAST 函数进行转换。

**%**：调制运算符。返回左操作数除以右操作数的余数。左操作数和右操作数都必须是整数。

**&、| 和 ^**：按位运算符返回对两个操作数的按位 AND、按位 OR、按位 XOR 运算的结果。这两个操作数都需要整数类型。

如果按位运算符的两个操作数的类型不一致，则将较小类型的操作数提升为较大类型的操作数，并执行相应的按位运算。

多个算术运算符可以出现在一个表达式中，用户可以将相应的算术表达式括在括号中。算术运算符通常没有相应的数学函数来表示与算术运算符相同的函数。

例如，我们没有 MOD() 函数来表示 % 运算符。相反，数学函数没有相应的算术运算符。例如，幂函数 POW() 没有相应的 ** 指数运算符。

用户可以通过“数学函数”部分了解我们支持哪些算术函数。

#### Between 运算符

在 where 子句中，表达式可以与上限和下限进行比较。如果表达式大于或等于下限且小于或等于上限，则比较结果为 true。

语法：

```sql
expression BETWEEN lower_bound AND upper_bound
```

数据类型：表达式的计算结果通常为数值类型，该类型也支持其他数据类型。如果必须确保下限和上限都是可比较的字符，则可以使用 cast() 函数。

使用说明：如果操作数为字符串类型，请注意，以上限开头的长字符串将不匹配上限，这大于上限。例如，“between 'A' and 'M' 将不匹配 'MJ'。

如果需要确保表达式正常工作，可以使用 upper()、lower()、substr()、trim() 等函数。

例子：

```sql
select c1 from t1 where month between 1 and 6;
```

#### 比较运算符

比较运算符用于比较两个值。`=`、`!=`、`>=` 适用于所有数据类型。

`<>` 和 `!=` 运算符是等效的，表示两个值不相等。

#### In 运算符

In 运算符与 VALUE 集合进行比较，如果它可以匹配集合中的任何元素，则返回 TRUE。

参数和 VALUE 集合必须具有可比性。所有使用 IN 运算符的表达式都可以写成与 OR 相关的等效比较，但 IN 的语法更简单、更精确，也更易于 StarRocks 优化。

例子：

```sql
select * from small_table where tiny_column in (1,2);
```

#### Like 运算符

此运算符用于与字符串进行比较。''匹配单个字符，'%'匹配多个字符。该参数必须与完整字符串匹配。通常，将'%'放在字符串的末尾更实用。

例子：

```plain text
mysql> select varchar_column from small_table where varchar_column like 'm%';

+----------------+
|varchar_column  |
+----------------+
|     milan      |
+----------------+

1 row in set (0.02 sec)
```

```plain
mysql> select varchar_column from small_table where varchar_column like 'm____';

+----------------+
| varchar_column | 
+----------------+
|    milan       | 
+----------------+

1 row in set (0.01 sec)
```

#### 逻辑运算符

逻辑运算符返回 BOOL 值，包括单位和多个运算符，每个运算符处理返回 BOOL 值的表达式的参数。支持的运算符有：

AND：2 元运算符，如果左侧和右侧的参数都计算为 TRUE，则 AND 运算符返回 TRUE。

OR：2 元运算符，如果左侧和右侧的参数之一计算为 TRUE，则 OR 运算符返回 TRUE。如果两个参数都为 FALSE，则 OR 运算符返回 FALSE。

NOT：单位运算符，反转表达式的结果。如果参数为 TRUE，则运算符返回 FALSE；如果参数为 FALSE，则运算符返回 TRUE。

例子：

```plain text
mysql> select true and true;

+-------------------+
| (TRUE) AND (TRUE) | 
+-------------------+
|         1         | 
+-------------------+

1 row in set (0.00 sec)
```

```plain text
mysql> select true and false;

+--------------------+
| (TRUE) AND (FALSE) | 
+--------------------+
|         0          | 
+--------------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select true or false;

+-------------------+
| (TRUE) OR (FALSE) | 
+-------------------+
|        1          | 
+-------------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select not true;

+----------+
| NOT TRUE | 
+----------+
|     0    | 
+----------+

1 row in set (0.01 sec)
```

#### 正则表达式运算符

确定正则表达式是否匹配。使用 POSIX 标准正则表达式，'^'匹配字符串的第一部分，'$'匹配字符串的末尾。

'.' 匹配任何单个字符，'*' 匹配零个或多个选项，'+' 匹配一个或多个选项，'?' 表示贪婪表示，依此类推。正则表达式需要匹配完整的值，而不仅仅是字符串的一部分。

如果要匹配中间部分，正则表达式的前面部分可以写成'^。'或'。'。'^' 和 '$' 通常被省略。RLIKE 运算符和 REGEXP 运算符是同义词。

'|' 运算符是可选运算符。'|' 两侧的正则表达式只需要满足一侧条件。'|' 运算符和两侧的正则表达式通常需要括在 () 中。

例子：

```plain text
mysql> select varchar_column from small_table where varchar_column regexp '(mi|MI).*';

+----------------+
| varchar_column | 
+----------------+
|     milan      |       
+----------------+


1 行记录(0.01 秒)
```

```plain text
mysql> 从 small_table 中选择 varchar_column，其中 varchar_column 匹配 'm.*';

+----------------+
| varchar_column | 
+----------------+
|     milan      |  
+----------------+

1 行记录(0.01 秒)
```

### 别名

在查询中写入包含列的表、列或表达式的名称时，可以为它们分配别名。别名通常比原始名称更短，更易记。

当需要别名时，只需在选择列表或 from 列表中的表、列和表达式名称后添加 AS 子句即可。AS 关键字是可选的。您也可以直接在原始名称后指定别名，而无需使用 AS。

如果别名或其他标识符与[StarRocks关键字](../keywords.md)同名，则需要将该名称用一对反引号括起来，例如 `rank`。

别名区分大小写。

例子：

```sql
从 big_table 中选择 tiny_column 作为 name, int_column 作为 sex;

从 big_table 中选择 sum(tiny_column) 作为 total_count;

从 small_table one, big_table two 中选择 one.tiny_column, two.int_column，其中 one.tiny_column = two.tiny_column;