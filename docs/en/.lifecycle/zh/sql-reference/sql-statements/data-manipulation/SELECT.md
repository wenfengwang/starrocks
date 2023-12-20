---
displayed_sidebar: English
---

# SELECT

## 描述

从一个或多个表、视图或物化视图查询数据。SELECT 语句通常包含以下子句：

- [WITH](#with)
- [WHERE 和 运算符](#where-and-operators)
- [GROUP BY](#group-by)
- [HAVING](#having)
- [UNION](#union)
- [INTERSECT](#intersect)
- [EXCEPT/MINUS](#exceptminus)
- [ORDER BY](#order-by)
- [LIMIT](#limit)
- [OFFSET](#offset)
- [Joins](#join)
- [Subqueries](#subquery)
- [DISTINCT](#distinct)
- [Alias](#alias)

SELECT 可以作为独立语句或嵌套在其他语句中的子句。SELECT 子句的输出可以用作其他语句的输入。

StarRocks 的查询语句基本符合 SQL92 标准。以下是对支持的 SELECT 用法的简要描述。

> **注意**
> 要从 StarRocks 内部表中的表、视图或物化视图查询数据，您必须对这些对象具有 SELECT 权限。要从外部数据源中的表、视图或物化视图查询数据，您必须具有相应外部目录的 USAGE 权限。

### WITH

可以在 SELECT 语句之前添加的子句，用于为 SELECT 内多次引用的复杂表达式定义别名。

类似于 CREATE VIEW，但子句中定义的表名和列名在查询结束后不会持久化，并且不会与实际表或 VIEW 中的名称冲突。

使用 WITH 子句的好处包括：

方便且易于维护，减少查询中的重复。

通过将查询中最复杂的部分抽象为单独的块，使 SQL 代码更易于阅读和理解。

示例：

```sql
-- 在外层定义一个子查询，在内层作为 UNION ALL 查询的初始阶段的一部分定义另一个子查询。

WITH t1 AS (SELECT 1), t2 AS (SELECT 2)
SELECT * FROM t1 UNION ALL SELECT * FROM t2;
```

### Join

Join 操作组合来自两个或多个表的数据，然后返回其中某些表中某些列的结果集。

StarRocks 支持自连接、交叉连接、内连接、外连接、半连接和反连接。外连接包括左连接、右连接和全连接。

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

#### Self Join

StarRocks 支持自连接，即同一个表的不同列之间的连接。

实际上没有特殊的语法来标识自连接。自连接中的连接条件两侧都来自同一个表。

我们需要为它们分配不同的别名。

示例：

```sql
SELECT lhs.id, rhs.parent, lhs.c1, rhs.c2 FROM tree_data lhs JOIN tree_data rhs ON lhs.id = rhs.parent;
```

#### Cross Join

交叉连接可能会产生大量结果，因此应谨慎使用交叉连接。

即使需要使用交叉连接，也需要使用过滤条件并确保返回较少的结果。示例：

```sql
SELECT * FROM t1 CROSS JOIN t2;
```

#### Inner Join

内连接是最知名和最常用的连接。返回两个相似表所请求的列的结果，如果两个表的列包含相同的值，则进行连接。

如果两个表的列名相同，我们需要使用全名（表名.列名的形式）或为列名指定别名。

示例：

以下三个查询是等效的。

```sql
SELECT t1.id, c1, c2 FROM t1 JOIN t2 WHERE t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```

#### Outer Join

外连接返回左表、右表或两者的所有行。如果另一个表中没有匹配的数据，则将其设置为 NULL。示例：

```sql
SELECT * FROM t1 LEFT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 RIGHT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;
```

#### Equi-join and Non-equi-join

通常，用户使用最多的是等值连接，要求连接条件的运算符为等号。

非等值连接可以使用连接条件 !=、<> 等。非等值连接可能会产生大量结果，并且在计算过程中可能会超出内存限制。

谨慎使用。非等值连接仅支持内连接。示例：

```sql
SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id > t2.id;
```

#### Semi Join

左半连接仅返回左表中与右表中的数据匹配的行，无论有多少行与右表中的数据匹配。

左表的这一行最多返回一次。右半连接的工作原理类似，只不过返回的数据是右表。

示例：

```sql
SELECT t1.c1, t1.c2 FROM t1 LEFT SEMI JOIN t2 ON t1.id = t2.id;
```

#### Anti Join

左反连接仅返回左表中与右表不匹配的行。

右反连接反转此比较，仅返回右表中与左表不匹配的行。示例：

```sql
SELECT t1.c1, t1.c2 FROM t1 LEFT ANTI JOIN t2 ON t1.id = t2.id;
```

#### Equi-join and Non-equi-join

StarRocks 支持的各种连接可以根据连接中指定的连接条件分为等值连接和非等值连接。

|**等值连接**|自连接、交叉连接、内连接、外连接、半连接和反连接|
|---|---|
|**非等值连接**|交叉连接、内连接、左半连接、左反连接和外连接|

- 等值连接

  等值连接使用连接条件，其中两个连接项通过 `=` 运算符组合。示例：`a JOIN b ON a.id = b.id`。

- 非等值连接

  非等值连接使用连接条件，其中两个连接项通过比较运算符如 `<`、`<=`、`>`、`>=` 或 `<>` 组合。示例：`a JOIN b ON a.id < b.id`。非等值连接的运行速度比等值连接慢。我们建议您在使用非等值连接时要小心。

  以下两个示例展示了如何运行非等值连接：

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

ORDER BY 是一种耗时且消耗资源的操作，因为必须将所有结果发送到一个节点进行合并，然后才能对结果进行排序。排序比没有 ORDER BY 的查询消耗更多的内存资源。

因此，如果您只需要排序结果集中的前 N 个结果，则可以使用 LIMIT 子句，这会减少内存使用和网络开销。如果未指定 LIMIT 子句，则默认返回前 65535 个结果。

语法：

```sql
ORDER BY <column_name> 
    [ASC | DESC]
    [NULLS FIRST | NULLS LAST]
```

`ASC` 指定结果应按升序返回。`DESC` 指定应按降序返回结果。如果未指定顺序，则默认为 ASC（升序）。示例：

```sql
SELECT * FROM big_table ORDER BY tiny_column, short_column DESC;
```

NULL 值的排序顺序：`NULLS FIRST` 表示 NULL 值应在非 NULL 值之前返回。`NULLS LAST` 表示应在非 NULL 值之后返回 NULL 值。

示例：

```sql
SELECT * FROM sales_record ORDER BY employee_id NULLS FIRST;
```

### GROUP BY

GROUP BY 子句通常与聚合函数（如 COUNT()、SUM()、AVG()、MIN() 和 MAX()）一起使用。

GROUP BY 指定的列不会参与聚合操作。GROUP BY 子句可以与 HAVING 子句一起添加，以过滤聚合函数产生的结果。

示例：

```sql
SELECT tiny_column, SUM(short_column)
FROM small_table 
GROUP BY tiny_column;
```

```plain
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

`groupSet` 表示由选择列表中的列、别名或表达式组成的集合。`groupSet ::= { ( expr [ , expr [ , ... ] ] )}`

`expr` 表示选择列表中的列、别名或表达式。

#### 注意

StarRocks 支持类似 PostgreSQL 的语法。语法示例如下：

```sql
SELECT a, b, SUM(c) FROM tab1 GROUP BY GROUPING SETS ((a, b), (a), (b), ());
SELECT a, b, c, SUM(d) FROM tab1 GROUP BY ROLLUP(a, b, c);
SELECT a, b, c, SUM(d) FROM tab1 GROUP BY CUBE(a, b, c);
```

`ROLLUP(a, b, c)` 相当于以下 `GROUPING SETS` 语句：

```sql
GROUPING SETS (
(a, b, c),
(a, b),
(a),
()
)
```

`CUBE (a, b, c)` 相当于以下 `GROUPING SETS` 语句：

```sql
GROUPING SETS (
(a, b, c),
(a, b),
(a, c),
(a),
(b, c),
(b),
(c),
()
)
```

#### 示例

以下为实际数据示例：

```plain
SELECT * FROM t;
+------+------+------+
```
mysql> (select id, price from select1) except (select id, price from select2)
order by id;
+------+-------+
| id   | price |
+------+-------+
|    1 |     2 |
+------+-------+

mysql> (select id, price from select1) minus (select id, price from select2)
order by id;
+------+-------+
| id   | price |
+------+-------+
|    1 |     2 |
+------+-------+
```sql
mysql> (SELECT id, price FROM select1) EXCEPT (SELECT id, price FROM select2) ORDER BY id;
+------+-------+
| id   | price |
+------+-------+
|    1 |     2 |
+------+-------+

mysql> (SELECT id, price FROM select1) MINUS (SELECT id, price FROM select2) ORDER BY id;
+------+-------+
| id   | price |
+------+-------+
|    1 |     2 |
+------+-------+
```

### DISTINCT

DISTINCT 关键字用于对结果集进行去重。示例：

```SQL
-- 返回一列中的唯一值。
SELECT DISTINCT tiny_column FROM big_table LIMIT 2;

-- 返回多列中的唯一值组合。
SELECT DISTINCT tiny_column, int_column FROM big_table LIMIT 2;
```

DISTINCT 可以与聚合函数（通常是 COUNT 函数）一起使用，COUNT(DISTINCT) 用于计算一列或多列中包含的不同值组合的数量。

```SQL
-- 计算一列中的唯一值数量。
SELECT COUNT(DISTINCT tiny_column) FROM small_table;
```

```plain
+-------------------------------+
| COUNT(DISTINCT tiny_column)   |
+-------------------------------+
|             2                 |
+-------------------------------+
1 row in set (0.06 sec)
```

```SQL
-- 计算多列中的唯一值组合数量。
SELECT COUNT(DISTINCT tiny_column, int_column) FROM big_table LIMIT 2;
```

StarRocks 支持同时使用多个带 DISTINCT 的聚合函数。

```SQL
-- 分别计算多个聚合函数的唯一值数量。
SELECT COUNT(DISTINCT tiny_column, int_column), COUNT(DISTINCT varchar_column) FROM big_table;
```

### 子查询

子查询根据相关性分为两类：

- 非相关子查询独立于其外部查询获取其结果。
- 相关子查询需要来自其外部查询的值。

#### 非相关子查询

非相关子查询支持 [NOT] IN 和 EXISTS。

示例：

```sql
SELECT x FROM t1 WHERE x [NOT] IN (SELECT y FROM t2);

SELECT * FROM t1 WHERE (x, y) [NOT] IN (SELECT x, y FROM t2 LIMIT 2);

SELECT x FROM t1 WHERE EXISTS (SELECT y FROM t2 WHERE y = 1);
```

从 v3.0 开始，您可以在 `SELECT...FROM...WHERE...[NOT] IN` 的 WHERE 子句中指定多个字段，例如第二个 SELECT 语句中的 `WHERE (x, y)`。

#### 相关子查询

相关子查询支持 [NOT] IN 和 [NOT] EXISTS。

示例：

```sql
SELECT * FROM t1 WHERE x [NOT] IN (SELECT a FROM t2 WHERE t1.y = t2.b);

SELECT * FROM t1 WHERE [NOT] EXISTS (SELECT a FROM t2 WHERE t1.y = t2.b);
```

子查询还支持标量子查询。它可以分为不相关的标量子查询、相关的标量子查询和作为通用函数参数的标量子查询。

示例：

1. 不相关的标量子查询，带有等于（=）谓词。例如，输出工资最高的人的信息。

   ```sql
   SELECT name FROM table WHERE salary = (SELECT MAX(salary) FROM table);
   ```

2. 不相关的标量子查询，带有大于（>）、小于（<）等谓词。例如，输出工资高于平均水平的人的信息。

   ```sql
   SELECT name FROM table WHERE salary > (SELECT AVG(salary) FROM table);
   ```

3. 相关的标量子查询。例如，输出每个部门最高工资的信息。

   ```sql
   SELECT name FROM table a WHERE salary = (SELECT MAX(salary) FROM table b WHERE b.Department = a.Department);
   ```

4. 标量子查询作为普通函数的参数。

   ```sql
   SELECT name FROM table WHERE salary = ABS((SELECT MAX(salary) FROM table));
   ```

### WHERE 和运算符

SQL 运算符是一系列用于比较的函数，广泛用于 SELECT 语句的 WHERE 子句中。

#### 算术运算符

算术运算符通常出现在包含左、右操作数的表达式中，最常见的是左操作数。

**+ 和 -**：可以用作一元或二元运算符。当用作一元运算符时，如 +1、-2.5 或 -col_name，意味着该值被乘以 +1 或 -1。

因此，一元运算符 + 返回未改变的值，而一元运算符 - 改变该值的符号位。

用户可以叠加两个一元运算符，例如 +5（返回正值）、-+2 或 +-2（返回负值），但用户不能使用两个连续的 - 符号。

因为 -- 在下面的语句中被解释为注释（当用户可以使用两个符号时，两个符号之间需要有空格或括号，例如 -(-2) 或 --2，这实际上会导致 +2）。

当 + 或 - 是二元运算符时，如 2+2、3+1.5 或 col1+col2，表示左值与右值相加或相减。左右值都必须是数值类型。

**\* 和 /**：分别代表乘法和除法。两边的操作数必须是数值类型。当两个数相乘时。

如果需要，较小的操作数可能会被提升（例如，SMALLINT 提升为 INT 或 BIGINT），并且表达式的结果将被提升为下一个更大的类型。

例如，TINYINT 乘以 INT 将产生 BIGINT 类型的结果。当两个数相乘时，操作数和表达式结果都被解释为 DOUBLE 类型以避免精度损失。

如果用户希望将表达式的结果转换为另一种类型，需要使用 CAST 函数进行转换。

**%**：取模运算符。返回左操作数除以右操作数的余数。左右操作数都必须是整数。

**&, | 和 ^**：按位运算符返回两个操作数的按位与、按位或、按位异或运算的结果。两个操作数都需要是整数类型。

如果按位运算符的两个操作数类型不一致，较小类型的操作数将被提升为较大类型的操作数，并执行相应的按位运算。

一个表达式中可以出现多个算术运算符，用户可以将相应的算术表达式括在括号中。算术运算符通常没有相应的数学函数来表达与算术运算符相同的功能。

例如，我们没有 MOD() 函数来表示 % 运算符。相反，数学函数没有相应的算术运算符。例如，幂函数 POW() 没有相应的 ** 求幂运算符。

用户可以通过数学函数部分了解我们支持哪些算术函数。

#### BETWEEN 运算符

在 WHERE 子句中，表达式可以与上限和下限进行比较。如果表达式大于或等于下限且小于或等于上限，则比较结果为真。

语法：

```sql
expression BETWEEN lower_bound AND upper_bound
```

数据类型：通常表达式计算结果为数值类型，也支持其他数据类型。如果必须确保下限和上限都是可比较的字符，则可以使用 CAST() 函数。

使用说明：如果操作数是字符串类型，请注意，以上限开头的长字符串将不会匹配上界，因为它大于上界。例如，“BETWEEN 'A' AND 'M'”不会匹配 'MJ'。

如果需要确保表达式正常工作，可以使用 UPPER()、LOWER()、SUBSTR()、TRIM() 等函数。

示例：

```sql
SELECT c1 FROM t1 WHERE month BETWEEN 1 AND 6;
```

#### 比较运算符

比较运算符用于比较两个值。`=`, `!=`, `>=` 适用于所有数据类型。

`<>` 和 `!=` 运算符是等效的，表示两个值不相等。

#### IN 运算符

IN 运算符与值集合进行比较，如果可以匹配集合中的任何元素，则返回 TRUE。

参数和值集合必须具有可比性。所有使用 IN 运算符的表达式都可以写成与 OR 连接的等价比较，但 IN 的语法更简单、更精确，也更易于 StarRocks 优化。

示例：

```sql
SELECT * FROM small_table WHERE tiny_column IN (1, 2);
```

#### LIKE 运算符

LIKE 运算符用于与字符串进行比较。`_` 匹配单个字符，`%` 匹配多个字符。参数必须匹配完整的字符串。通常，将 `%` 放在字符串末尾更为实用。

示例：

```sql
SELECT varchar_column FROM small_table WHERE varchar_column LIKE 'm%';
```

```plain
+----------------+
| varchar_column |
+----------------+
|     milan      |
+----------------+

1 row in set (0.02 sec)
```

```sql
SELECT varchar_column FROM small_table WHERE varchar_column LIKE 'm____';
```

```plain
+----------------+
| varchar_column |
+----------------+
|     milan      |
+----------------+

1 row in set (0.01 sec)
```

#### 逻辑运算符

逻辑运算符返回一个 BOOL 值，包括一元和多元运算符，每个运算符处理的参数是返回 BOOL 值的表达式。支持的运算符有：

AND：二元运算符，如果左右参数计算结果均为 TRUE，则 AND 运算符返回 TRUE。

OR：二元运算符，如果左右参数之一计算为 TRUE，则返回 TRUE。如果两个参数均为 FALSE，则 OR 运算符返回 FALSE。

NOT：一元运算符，结果为表达式取反。如果参数为 TRUE，则运算符返回 FALSE；如果参数为 FALSE，则运算符返回 TRUE。

示例：

```sql
SELECT TRUE AND TRUE;
```

```plain
+-------------------+
| (TRUE) AND (TRUE) |
+-------------------+
|         1         |
+-------------------+

1 row in set (0.00 sec)
```

```sql
SELECT TRUE AND FALSE;
```

```plain
+--------------------+
| (TRUE) AND (FALSE) |
+--------------------+
|         0          |
+--------------------+

1 row in set (0.01 sec)
```

```sql
SELECT TRUE OR FALSE;
```

```plain
+-------------------+
| (TRUE) OR (FALSE) |
+-------------------+
|        1          |
+-------------------+

1 row in set (0.01 sec)
```

```sql
SELECT NOT TRUE;
```

```plain
+----------+
| NOT TRUE |
+----------+
|     0    |
+----------+

1 row in set (0.01 sec)
```

#### 正则表达式运算符

判断正则表达式是否匹配。使用 POSIX 标准正则表达式，`^` 匹配字符串的开头，`$` 匹配字符串的结尾。

`.` 匹配任何单个字符，`*` 匹配零个或多个字符，`+` 匹配一个或多个字符，`?` 表示贪婪匹配，等等。正则表达式需要匹配完整的值，而不仅仅是字符串的一部分。

如果要匹配中间部分，正则表达式的前面可以写成 `^. *` 或 `.*`。`^` 和 `$` 通常被省略。RLIKE 运算符和 REGEXP 运算符是同义词。

`|` 运算符是可选运算符。`|` 两侧的正则表达式只需要满足一个条件即可。`|` 运算符和两侧的正则表达式通常需要用括号 `()` 括起来。

示例：

```sql
SELECT varchar_column FROM small_table WHERE varchar_column REGEXP '(mi|MI).*';
```

```plain
+----------------+
| varchar_column |
+----------------+
|     milan      |
+----------------+

1 row in set (0.01 sec)
```
```plain
mysql> select varchar_column from small_table where varchar_column regexp 'm.*';

+----------------+
| varchar_column | 
+----------------+
|     milan      |  
+----------------+

1 row in set (0.01 sec)
```

### 别名

当您在查询中编写表、列或包含列的表达式的名称时，可以为它们指定别名。别名通常比原始名称更短、更易记忆。

需要别名时，您可以在 select 列表或 from 列表中的表、列和表达式名称之后直接添加 AS 子句。AS 关键字是可选的。您也可以在原始名称之后直接指定别名，无需使用 AS。

如果别名或其他标识符与内部 [StarRocks 关键字](../keywords.md) 同名，则需要将该名称用一对反引号括起来，例如 `rank`。

别名是区分大小写的。

示例：

```sql
select tiny_column as name, int_column as sex from big_table;

select sum(tiny_column) as total_count from big_table;

select one.tiny_column, two.int_column from small_table one, big_table two where one.tiny_column = two.tiny_column;
```