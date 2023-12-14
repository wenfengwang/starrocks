---
displayed_sidebar: "Chinese"
---

# array_agg

## 描述

将某一列的值（包括`NULL`）聚合为一个数组（多行到一行），并可选择按照特定列对元素进行排序。从v3.0开始，array_agg()支持使用ORDER BY对元素进行排序。

## 语法

```Haskell
ARRAY_AGG([distinct] col [order by col0 [desc | asc] [nulls first | nulls last] ...])
```

## 参数

- `col`: 想要聚合的列。支持的数据类型有BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、ARRAY (v3.1开始)、MAP (v3.1开始)和STRUCT (v3.1开始)。

- `col0`: 决定`col`排序的列。可以有多个ORDER BY列。

- `[desc | asc]`: 指定按照`col0`的升序（默认）或降序进行排序。

- `[nulls first | nulls last]`: 指定空值是放在最前面还是最后面。

## 返回值

返回一个ARRAY类型的值，可选择按照`col0`排序。

## 用法注意事项

- 数组中元素的顺序是随机的，这意味着如果没有指定ORDER BY列或按照ORDER BY列排序，则可能与列中的值顺序不同。
- 返回数组中元素的数据类型与列中值的数据类型相同。
- 如果输入为空且没有分组列，则返回`NULL`。

## 示例

以以下数据表为例：

```plaintext
mysql> select * from t;
+------+------+------+
| a    | name | pv   |
+------+------+------+
|   11 |      |   33 |
| 2    | NULL |  334 |
|    1 | fzh  |    3 |
|    1 | fff  |    4 |
|    1 | fff  |    5 |
+------+------+------+
```

示例1：按照`name`的顺序，对列`a`中的值进行分组，并将列`pv`的值聚合为一个数组。

```plaintext
mysql> select a, array_agg(pv order by name nulls first) from t group by a;
+------+---------------------------------+
| a    | array_agg(pv ORDER BY name ASC) |
+------+---------------------------------+
| 2    | [334]                           |
|   11 | [33]                            |
|    1 | [4,5,3]                         |
+------+---------------------------------+

-- 没有排序情况下聚合值。
mysql> select a, array_agg(pv) from t group by a;
+------+---------------+
| a    | array_agg(pv) |
+------+---------------+
|   11 | [33]          |
| 2    | [334]         |
|    1 | [3,4,5]       |
+------+---------------+
3 rows in set (0.03 sec)
```

示例2：按照`name`的顺序，将列`pv`的值聚合为一个数组。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| [3,4,5,33,334]                   |
+----------------------------------+
1 row in set (0.02 sec)

-- 没有排序情况下聚合值。
mysql> select array_agg(pv) from t;
+----------------+
| array_agg(pv)  |
+----------------+
| [3,4,5,33,334] |
+----------------+
1 row in set (0.03 sec)
```

示例3：使用WHERE子句对列`pv`的值进行聚合。如果`pv`中没有符合过滤条件的数据，则返回`NULL`值。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t where a < 0;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| NULL                             |
+----------------------------------+
1 row in set (0.02 sec)

-- 没有排序情况下聚合值。
mysql> select array_agg(pv) from t where a < 0;
+---------------+
| array_agg(pv) |
+---------------+
| NULL          |
+---------------+
1 row in set (0.03 sec)
```

## 关键字

ARRAY_AGG, ARRAY