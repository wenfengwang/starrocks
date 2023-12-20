---
displayed_sidebar: English
---

# array_agg

## 描述

将列中的值（包括 `NULL`）聚合成一个数组（多行合并为一行），并可选择按特定列对元素进行排序。从 v3.0 开始，`array_agg()` 支持使用 `ORDER BY` 对元素进行排序。

## 语法

```Haskell
ARRAY_AGG([distinct] col [order by col0 [desc | asc] [nulls first | nulls last] ...])
```

## 参数

- `col`：要聚合其值的列。支持的数据类型包括 BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、ARRAY（自 v3.1 起）、MAP（自 v3.1 起）和 STRUCT（自 v3.1 起）。

- `col0`：决定 `col` 排序的列。可能有多个 `ORDER BY` 列。

- `[desc | asc]`：指定是否按照 `col0` 的升序（默认）或降序对元素进行排序。

- `[nulls first | nulls last]`：指定空值是放在首位还是末位。

## 返回值

返回一个 `ARRAY` 类型的值，可选择按 `col0` 排序。

## 使用说明

- 数组中元素的顺序是随机的，这意味着如果未指定 `ORDER BY` 列或未按 `ORDER BY` 列排序，则数组中元素的顺序可能与列中值的顺序不同。
- 返回数组中元素的数据类型与列中值的数据类型相同。
- 如果输入为空且没有分组依据列，则返回 `NULL`。

## 示例

以以下数据表为例：

```plaintext
mysql> select * from t;
+------+------+------+
| a    | name | pv   |
+------+------+------+
|   11 |      |   33 |
|    2 | NULL |  334 |
|    1 | fzh  |    3 |
|    1 | fff  |    4 |
|    1 | fff  |    5 |
+------+------+------+
```

示例 1：按 `name` 排序，将 `a` 列中的值分组并将 `pv` 列中的值聚合成数组。

```plaintext
mysql> select a, array_agg(pv order by name nulls first) from t group by a;
+------+---------------------------------+
| a    | array_agg(pv ORDER BY name ASC) |
+------+---------------------------------+
|    2 | [334]                           |
|   11 | [33]                            |
|    1 | [4,5,3]                         |
+------+---------------------------------+

-- 无排序聚合值。
mysql> select a, array_agg(pv) from t group by a;
+------+---------------+
| a    | array_agg(pv) |
+------+---------------+
|   11 | [33]          |
|    2 | [334]         |
|    1 | [3,4,5]       |
+------+---------------+
3 rows in set (0.03 sec)
```

示例 2：将 `pv` 列中的值聚合成数组，并按 `name` 降序排序。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| [3,4,5,33,334]                   |
+----------------------------------+
1 row in set (0.02 sec)

-- 无排序聚合值。
mysql> select array_agg(pv) from t;
+----------------+
| array_agg(pv)  |
+----------------+
| [3,4,5,33,334] |
+----------------+
1 row in set (0.03 sec)
```

示例 3：使用 `WHERE` 子句聚合 `pv` 列中的值。如果没有 `pv` 数据满足过滤条件，则返回 `NULL` 值。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t where a < 0;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| NULL                             |
+----------------------------------+
1 row in set (0.02 sec)

-- 无排序聚合值。
mysql> select array_agg(pv) from t where a < 0;
+---------------+
| array_agg(pv) |
+---------------+
| NULL          |
+---------------+
1 row in set (0.03 sec)
```

## 关键词

`ARRAY_AGG`，`ARRAY`