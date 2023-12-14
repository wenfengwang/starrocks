---
displayed_sidebar: "Chinese"
---

# SELECT

## 描述

从一个或多个表、视图或物化视图中查询数据。SELECT语句通常由以下子句组成：

- [WITH](#with)
- [WHERE and operators](#where-and-operators)
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

SELECT可以作为独立语句或嵌套在其他语句中的子句。SELECT子句的输出可以作为其他语句的输入。

StarRocks的查询语句基本符合SQL92标准。以下是对支持的SELECT用法的简要描述。

> **注意**
>
> 要从StarRocks内部表中的表、视图或物化视图中查询数据，必须对这些对象具有SELECT权限。 要从外部数据源中的表、视图或物化视图中查询数据，必须对相应外部目录具有USAGE权限。

### WITH

可以在SELECT语句之前添加的子句，用于为在SELECT中多次引用的复杂表达式定义别名。

类似于CRATE VIEW，但是在查询结束后，子句中定义的表和列名不会持久化，并且不会与实际表或视图中的名称发生冲突。

使用WITH子句的好处包括：

方便且易于维护，减少查询中的重复。

通过将查询的最复杂的部分抽象成单独的块，能够更轻松地阅读和理解SQL代码。

示例：

```sql
-- 在UNION ALL查询的初始阶段，在外部级别定义一个子查询，在内部级别定义另一个子查询。

with t1 as (select 1), t2 as (select 2)
select * from t1 union all select * from t2;
```

### Join

连接操作将来自两个或多个表的数据组合在一起，然后返回其中一些表的某些列的结果集。

StarRocks支持自连接、交叉连接、内连接、外连接、半连接和反连接。外连接包括左连接、右连接和全连接。

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

StarRocks支持自连接，即自连接和自连接。例如，加入同一表的不同列。

实际上，没有特殊语法来识别自连接。自连接中连接的两侧的条件来自同一表。

我们需要为它们分配不同的别名。

示例:

```sql
SELECT lhs.id, rhs.parent, lhs.c1, rhs.c2 FROM tree_data lhs, tree_data rhs WHERE lhs.id = rhs.parent;
```

#### 交叉连接

交叉连接可以产生大量的结果，因此应谨慎使用交叉连接。

即使需要使用交叉连接，也需要使用过滤条件并确保返回较少的结果。 例如：

```sql
SELECT * FROM t1, t2;

SELECT * FROM t1 CROSS JOIN t2;
```

#### 内连接

内连接是最著名且最常用的连接。如果两个相似的表的列包含相同的值，则返回由两个表请求的结果。

如果两个表的列名相同，我们需要使用全名（以表名.列名的形式）或为列名取别名。

示例：

下面的三个查询是等价的。

```sql
SELECT t1.id, c1, c2 FROM t1, t2 WHERE t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```

#### 外连接

外连接返回左表或右表或两者的所有行。如果另一个表中没有匹配的数据，则设置为NULL。 例如：

```sql
SELECT * FROM t1 LEFT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 RIGHT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;
```

#### 等效和不等效连接

通常用户使用最等效连接，这要求连接条件的运算符是一个等号。

不等连接可以用于连接条件!=，等号。 不等连接会生成大量结果，并且在计算过程中可能超出内存限制。

要谨慎使用。 不等连接仅支持内连接。 例如：

```sql
SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id > t2.id;
```

#### 半连接

左半连接仅返回左表中与右表匹配的行，而不管右表匹配行数是多少。

返回左表的这一行最多返回一次。 右半连接类似工作，只是返回的数据是右表的。

示例：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT SEMI JOIN t2 ON t1.id = t2.id;
```

#### 反连接

左反连接仅返回左表中不匹配右表的行。

右反连接颠倒这种比较，仅返回右表中不匹配左表的行。 例如：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT ANTI JOIN t2 ON t1.id = t2.id;
```

#### 等值连接和非等值连接

StarRocks支持的各种连接可以根据连接中指定的连接条件分类为等值连接和非等值连接。

| **等值****连接**       | 自连接、交叉连接、内连接、外连接、半连接和反连接   |
| -------------------------- | ------------------------------------------------------------ |
| **非等值****连接** | 交叉连接、内连接、左半连接、左反连接和外连接            |

- 等值连接
  
  等值连接使用连接条件，其中两个连接项由`=`运算符组合。 例如： `a JOIN b ON a.id = b.id`。

- 非等值连接
  
  非等值连接使用连接条件，其中两个连接项由`<`， `<=`， `>`，`>=` 或 `<>`等比较运算符组合。 例如： `a JOIN b ON a.id < b.id`。 非等值连接比等值连接运行速度慢。 我们建议您在使用非等值连接时要小心。

  下面两个示例展示了如何运行非等值连接：

  ```SQL
  SELECT t1.id, c1, c2 
  FROM t1 
  INNER JOIN t2 ON t1.id < t2.id;
    
  SELECT t1.id, c1, c2 
  FROM t1 
  LEFT JOIN t2 ON t1.id > t2.id;
  ```

### ORDER BY

SELECT语句的ORDER BY子句通过比较一个或多个列的值来对结果集进行排序。

ORDER BY是一个耗时且资源消耗大的操作，因为所有结果必须发送到一个节点进行合并，然后才能对结果进行排序。 排序比没有ORDER BY子句的查询消耗更多的内存资源。

因此，如果只需要对排序后的结果集中的前`N`个结果，可以使用LIMIT子句，这样可以减少内存使用量和网络开销。 如果未指定LIMIT子句，默认返回前65535个结果。

语法：

```sql
ORDER BY <column_name> 
```
```sql
    + {R}
    + {R}
  + {R}
```

```sql
    + {T}
    + {T}
  + {T}
```
```plain text
mysql> select tiny_column from small_table limit 10000;

+-------------+
|tiny_column  |

+-------------+
|      1      |

|      2      |
+-------------+

2 rows in set (0.01 sec)
```


### 偏移量


OFFSET 子句导致结果集跳过前几行并直接返回以下结果。

默认情况下，结果集从第 0 行开始，因此偏移 0 和无偏移返回相同的结果。

一般来说，偏移量子句需要与 ORDER BY 和 LIMIT 子句一起使用才能生效。

示例：

```plain text

mysql> select varchar_column from big_table order by varchar_column limit 3;


+----------------+
| varchar_column | 
+----------------+
|    beijing     | 
|    chongqing   | 
|    tianjin     | 
+----------------+

3 rows in set (0.02 sec)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 0;

+----------------+
|varchar_column  |
+----------------+
|     beijing    |
+----------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 1;

+----------------+
|varchar_column  |
+----------------+
|    chongqing   | 
+----------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 2;

+----------------+
|varchar_column  |
+----------------+
|     tianjin    |     
+----------------+

1 row in set (0.02 sec)
```

注意：允许在没有 ORDER BY 的情况下使用偏移语法，但此时偏移量没有意义。

在这种情况下，只用取 limit 值，偏移值将被忽略。所以没有 order by。

偏移量超出结果集中的最大行数，仍然是一个结果。建议用户使用带有 order by 的偏移。

### 合并

合并多个查询的结果。

**语法:**

```sql
query_1 UNION [DISTINCT | ALL] query_2
```

- DISTINCT（默认）返回唯一的行。UNION 等效于 UNION DISTINCT。
- ALL 组合所有行，包括重复项。由于去重需要大量内存，使用 UNION ALL 的查询速度更快，占用的内存更少。为了获得更好的性能，请使用 UNION ALL。

> **注意**
>
> 每个查询语句必须返回相同数量的列，且列必须具有兼容的数据类型。

**示例:**

创建表 `select1` 和 `select2`。

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

示例 1：返回两张表中的所有 ID，包括重复项。

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
11 rows in set (0.02 sec)
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
6 rows in set (0.01 sec)

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
5 rows in set (0.02 sec)
```

示例 3：返回两个表中所有唯一 ID 中的前三个。以下两个语句是等效的。

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
4 rows in set (0.11 sec)

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
3 rows in set (0.01 sec)
```

### **交集**

计算多个查询结果的交集，即出现在所有结果集中的结果。此子句只返回唯一行，不支持 ALL 关键字。

**语法:**

```SQL
query_1 INTERSECT [DISTINCT] query_2
```

> **注意**
>
> - INTERSECT 等效于 INTERSECT DISTINCT。
> - 每个查询语句必须返回相同数量的列，且列必须具有兼容的数据类型。

**示例:**

UNION 中的两个表被使用。

返回两张表中共同出现的唯一 `(id, price)` 组合。以下两个语句是等效的。

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

### **差集**

返回在左侧查询中唯一出现且右侧查询中不存在的结果。EXCEPT 等效于 MINUS。

**语法:**

```SQL
query_1 {EXCEPT | MINUS} [DISTINCT] query_2
```

> **注意**
>
> - EXCEPT 等效于 EXCEPT DISTINCT。不支持 ALL 关键字。
> - 每个查询语句必须返回相同数量的列，且列必须具有兼容的数据类型。

**示例:**

UNION 中的两个表被使用。

返回在 `select1` 中出现但在 `select2` 中找不到的唯一 `(id, price)` 组合。

```Plaintext
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
```

### 唯一

DISTINCT 关键字对结果集去重。示例：

```SQL
-- 返回一列的唯一值。
select distinct tiny_column from big_table limit 2;

-- 返回多列的唯一值组合。
select distinct tiny_column, int_column from big_table limit 2;
```

DISTINCT 可以与聚合函数（通常是 count 函数）一起使用，count(distinct) 用于计算一列或多列上包含多少个不同的组合。

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
-- 计算多列的唯一值组合。
select count(distinct tiny_column, int_column) from big_table limit 2;
```

StarRocks 支持在同一时间使用多个带有 distinct 的聚合函数。

```SQL
-- 分别计算多个聚合函数的唯一值。
select count(distinct tiny_column, int_column), count(distinct varchar_column) from big_table;
```

### 子查询

子查询根据相关性可分为两种类型：

- 非相关子查询独立获取其结果。
- 相关子查询需要来自外部查询的值。

#### 调用频繁的子查询

不相关的子查询支持[NOT] IN和EXISTS。

例子:

```sql
SELECT x FROM t1 WHERE x [NOT] IN (SELECT y FROM t2);

SELECT * FROM t1 WHERE (x,y) [NOT] IN (SELECT x,y FROM t2 LIMIT 2);

SELECT x FROM t1 WHERE EXISTS (SELECT y FROM t2 WHERE y = 1);
```

从版本3.0开始，您可以在`SELECT... FROM... WHERE... [NOT] IN`的WHERE子句中指定多个字段，例如第二个SELECT语句中的`WHERE (x,y)`。

#### 相关的子查询

相关的子查询支持[NOT] IN和[NOT] EXISTS。

例子:

```sql
SELECT * FROM t1 WHERE x [NOT] IN (SELECT a FROM t2 WHERE t1.y = t2.b);

SELECT * FROM t1 WHERE [NOT] EXISTS (SELECT a FROM t2 WHERE t1.y = t2.b);
```

子查询还支持标量量子查询。它可以分为不相关的标量量子查询，相关的标量量子查询和作为通用函数参数的标量量子查询。

例子:

1. 使用带有=符号的不相关标量量子查询。例如，输出有最高工资的人的信息。

    ```sql
    SELECT name FROM table WHERE salary = (SELECT MAX(salary) FROM table);
    ```

2. 使用带有`>`、`<`等的不相关标量量子查询。例如，输出工资高于平均工资的人的信息。

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

### WHERE和操作符

SQL操作符是一系列用于比较的函数，广泛用于select语句的where子句中。

#### 算术操作符

算术操作符通常出现在包含左、右和最常见的左操作数的表达式中

**+和-**: 可以作为一个单位或二元运算符使用。当作为单位运算符使用时，比如+1，-2.5或-col_ name，意思是该值乘以+1或-1。

所以单元运算符+返回一个不变的值，单元运算符-改变该值的符号位。

用户可以叠加两个单元运算符，比如+5（返回正值），-+2或+2（返回负值），但用户不能使用两个连续的-号。

因为--在下面的语句中被解释为注释（当用户可以使用两个符号时，两个符号之间需要空格或括号，比如-(-2)或- -2，实际上结果是+2）。

当+或-是二元运算符时，比如2+2，3+1.5，或col1+col2，它表示左值与右值相加或相减。左右值都必须是数值类型。

**和/**: 分别表示乘法和除法。两侧的操作数必须是数据类型。当两个数字相乘。

如果需要，可以提升较小的操作数（例如，从SMALLINT到INT或BIGINT），并且表达式的结果将被提升到下一个更大的类型。

例如，TINYINT乘以INT将产生BIGINT类型的结果。当两个数字相乘时，操作数和表达式结果都被解释为DOUBLE类型，以避免精度损失。

如果用户希望转换表达式的结果为另一种类型，需要使用CAST函数进行转换。

**%**: 模运算符。返回左操作数除以右操作数的余数。左右操作数都必须是整数。

**&, |和^**: 位操作符返回两个操作数的位AND、位OR、位XOR操作的结果。两个操作数都需要整数类型。

如果位操作符的两个操作数的类型不一致，小类型的操作数将被提升为大类型的操作数，并执行相应的位操作。

多个算术运算符可以出现在表达式中，用户可以将对应的算术表达式括在括号中。算术运算符通常没有与算术运算符相同的数学函数来表示相同的函数。

例如，我们没有MOD()函数来表示%运算符。相反，数学函数没有对应的算术运算符。例如，幂函数POW()没有对应的**幂运算符。

用户可以通过数学函数部分找出我们支持哪些算术函数。

#### Between操作符

在where子句中，表达式可以与上限和下限进行比较。如果表达式大于或等于下限，且小于或等于上限，则比较的结果为真。

语法:

```sql
expression BETWEEN lower_bound AND upper_bound
```

数据类型: 通常表达式会评估为数值类型，也支持其他数据类型。如果必须确保下限和上限都是可比较的字符，可以使用cast()函数。

使用说明: 如果操作数是字符串类型，注意以大写字母开头的长字符串将不能匹配上限大于上限的情况。例如，"between'A'and'M'不匹配'MJ'。

如果需要确保表达式正常工作，可以使用upper()、lower()、substr()、trim()等函数。

例子:

```sql
select c1 from t1 where month between 1 and 6;
```

#### 比较操作符

比较操作符用于比较两个值。`=`、`!=`、`>=`适用于所有数据类型。

`<>`和`!=`运算符是等效的，表示两个值不相等。

#### In操作符

In操作符与VALUE集合进行比较，如果能匹配集合中的任何元素，则返回真。

参数和VALUE集合必须是可比较的。使用In操作符的所有表达式都可以写为与OR连接的等效比较，但是In的语法更简单、更精确，并且更容易优化。

例子:

```sql
select * from small_table where tiny_column in (1,2);
```

#### Like操作符

此操作符用于对字符串进行比较。''匹配单个字符，'%'匹配多个字符。参数必须匹配完整的字符串。通常，在字符串末尾放置'%'更实用。

例子:

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

#### 逻辑操作符

逻辑操作符返回一个BOOL值，包括单元和多元操作符，每个操作符处理返回BOOL值的表达式参数。支持的操作符包括:

AND: 二元操作符，AND操作符如果左右参数都计算为TRUE则返回TRUE。

OR: 二元操作符，如果左右参数中有一个计算为TRUE则返回TRUE。如果两个参数都为FALSE，OR操作符返回FALSE。

NOT: 单元操作符，表示对表达式的结果进行取反。如果参数为TRUE，则操作符返回FALSE；如果参数为FALSE，则操作符返回TRUE。

例子:

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

#### 正则表达式操作符

确定是否匹配正则表达式。使用POSIX标准正则表达式，'^'匹配字符串的开头部分，'$'匹配字符串的结尾部分。

"."匹配任意单个字符，"*"匹配零个或多个选项，"+"匹配一个或多个选项，"?"表示贪婪表示，等等。正则表达式需要匹配完整的值，而不仅仅是字符串的部分。

如果要匹配中间部分，正则表达式的前部分可写为'^。'或'.'。'^'和'$'通常被省略。RLIKE 运算符和 REGEXP 运算符是同义词。

“|”运算符是可选运算符。'|'两侧的正则表达式只需要满足一侧条件即可。“|”运算符和两侧的正则表达式通常需要用 () 括起来。

示例：

```plain text
mysql> select varchar_column from small_table where varchar_column regexp '(mi|MI).*';

+----------------+
| varchar_column | 
+----------------+
|     milan      |       
+----------------+

1 行受影响 (0.01 秒)
```

```plain text
mysql> select varchar_column from small_table where varchar_column regexp 'm.*';

+----------------+
| varchar_column | 
+----------------+
|     milan      |  
+----------------+

1 行受影响 (0.01 秒)
```

### 别名

在查询中编写表、列或包含列的表达式的名称时，可以为它们指定别名。别名通常比原始名称更简短，更容易记忆。

需要别名时，可以在 select 列表或 from 列表中的表、列和表达式名称后简单地添加 AS 子句。AS 关键字是可选的。您也可以直接在原始名称之后指定别名，而不使用 AS。

如果别名或其他标识符与内部 [StarRocks 关键字](../keywords.md) 同名，则需要将名称括在一对反引号中，例如，`rank`。

别名区分大小写。

示例：

```sql
select tiny_column as name, int_column as sex from big_table;

select sum(tiny_column) as total_count from big_table;

select one.tiny_column, two.int_column from small_table one, <br/> big_table two where one.tiny_column = two.tiny_column;
```