---
displayed_sidebar: English
---

# 数组聚合

## 描述

将列中的值（包括 NULL）聚合成一个数组（多行合并为一行），并且可以按特定列对元素进行排序。从 v3.0 版本开始，array_agg() 支持使用 ORDER BY 来对元素进行排序。

## 语法

```Haskell
ARRAY_AGG([distinct] col [order by col0 [desc | asc] [nulls first | nulls last] ...])
```

## 参数

- col：你想要聚合的列的值。支持的数据类型有 BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、ARRAY（从 v3.1 版本起）、MAP（从 v3.1 版本起）和 STRUCT（从 v3.1 版本起）。

- col0：用于决定 col 排序的列。可以有多个 ORDER BY 列。

- [desc | asc]：指定按 col0 的升序（默认）或降序来排序元素。

- [nulls first | nulls last]：指定空值是放在开头还是末尾。

## 返回值

返回 ARRAY 类型的值，可选择按 col0 排序。

## 使用说明

- 如果没有指定 ORDER BY 列或没有按 ORDER BY 列排序，则数组中元素的顺序是随机的，这意味着它可能与列中值的顺序不同。
- 返回数组中元素的数据类型与列中值的数据类型一致。
- 如果输入为空并且没有分组列，则返回 NULL。

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

示例 1：按名称对 a 列中的值进行分组，并按照该顺序将 pv 列中的值聚合成一个数组。

```plaintext
mysql> select a, array_agg(pv order by name nulls first) from t group by a;
+------+---------------------------------+
| a    | array_agg(pv ORDER BY name ASC) |
+------+---------------------------------+
|    2 | [334]                           |
|   11 | [33]                            |
|    1 | [4,5,3]                         |
+------+---------------------------------+

-- Aggregate values with no order.
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

示例 2：将 pv 列中的值聚合成一个数组，并按名称进行排序。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| [3,4,5,33,334]                   |
+----------------------------------+
1 row in set (0.02 sec)

-- Aggregate values with no order.
mysql> select array_agg(pv) from t;
+----------------+
| array_agg(pv)  |
+----------------+
| [3,4,5,33,334] |
+----------------+
1 row in set (0.03 sec)
```

示例 3：使用 WHERE 子句聚合 pv 列中的值。如果 pv 中没有数据满足过滤条件，则返回 NULL 值。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t where a < 0;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| NULL                             |
+----------------------------------+
1 row in set (0.02 sec)

-- Aggregate values with no order.
mysql> select array_agg(pv) from t where a < 0;
+---------------+
| array_agg(pv) |
+---------------+
| NULL          |
+---------------+
1 row in set (0.03 sec)
```

## 关键词

ARRAY_AGG，数组聚合
