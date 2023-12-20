---
displayed_sidebar: English
---

# GROUP BY

## 描述

GROUP BY `GROUPING SETS` | `CUBE` | `ROLLUP` 是 GROUP BY 子句的扩展。它可以实现在 GROUP BY 子句中对多个集合的组进行聚合。结果相当于多个相应 GROUP BY 子句的 UNION 操作。

GROUP BY 子句是只包含一个元素的 GROUP BY `GROUPING SETS` 的特殊情况。例如，GROUPING SETS 语句：

```sql
SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
```

查询结果相当于：

```sql
SELECT a, b, SUM( c ) FROM tab1 GROUP BY a, b
UNION
SELECT a, NULL, SUM( c ) FROM tab1 GROUP BY a
UNION
SELECT NULL, b, SUM( c ) FROM tab1 GROUP BY b
UNION
SELECT NULL, NULL, SUM( c ) FROM tab1
```

`GROUPING(expr)` 表示列是否为聚合列。如果是聚合列，则为 0，否则为 1。

`GROUPING_ID(expr [, expr [, ... ]])` 类似于 GROUPING。GROUPING_ID 根据指定的列顺序计算列列表的位图值，每一位都是 GROUPING 的值。

GROUPING_ID() 函数返回位向量的十进制值。

### 语法

```sql
SELECT ...
FROM ...
[ ... ]
GROUP BY [
    , ... |
    GROUPING SETS [, ...] ( groupSet [, groupSet [, ... ]] ) |
    ROLLUP(expr [, expr [, ... ]]) |
    expr [, expr [, ... ]] WITH ROLLUP |
    CUBE(expr [, expr [, ... ]]) |
    expr [, expr [, ... ]] WITH CUBE
]
[ ... ]
```

### 参数

`groupSet` 表示由选择列表中的列、别名或表达式组成的集合。`groupSet ::= { ( expr [, expr [, ... ]] ) }`

`expr` 表示选择列表中的列、别名或表达式。

### 注意

StarRocks 支持类似 PostgreSQL 的语法。语法示例如下：

```sql
SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
SELECT a, b, c, SUM( d ) FROM tab1 GROUP BY ROLLUP(a, b, c)
SELECT a, b, c, SUM( d ) FROM tab1 GROUP BY CUBE(a, b, c)
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

`CUBE(a, b, c)` 相当于以下 `GROUPING SETS` 语句：

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

## 示例

以下是实际数据的示例：

```plain
> SELECT * FROM t;
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
8 行记录集 (0.01 秒)

> SELECT k1, k2, SUM(k3) FROM t GROUP BY GROUPING SETS ( (k1, k2), (k2), (k1), ( ) );
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
9 行记录集 (0.06 秒)

> SELECT k1, k2, GROUPING_ID(k1, k2), SUM(k3) FROM t GROUP BY GROUPING SETS ((k1, k2), (k1), (k2), ());
+------+------+---------------------+-----------+
| k1   | k2   | GROUPING_ID(k1, k2) | sum(`k3`) |
+------+------+---------------------+-----------+
| a    | A    |                   0 |         3 |
| a    | B    |                   0 |         4 |
| a    | NULL |                   1 |         7 |
| b    | A    |                   0 |         5 |
| b    | B    |                   0 |         6 |
| b    | NULL |                   1 |        11 |
| NULL | A    |                   2 |         8 |
| NULL | B    |                   2 |        10 |
| NULL | NULL |                   3 |        18 |
+------+------+---------------------+-----------+
9 行记录集 (0.02 秒)
```