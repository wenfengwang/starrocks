---
displayed_sidebar: "Chinese"
---

# 分组

## 描述

指示列是否是聚合列。如果是聚合列，则返回0。否则返回1。

## 语法

```Haskell
GROUPING(col_expr)
```

## 参数

`col_expr`: 在GROUP BY子句的ROLLUP、CUBE或GROUPING SETS扩展的表达式列表中指定的维度列的表达式。

## 示例

```plain text
MySQL > select * from t;
+------+------+
| k1   | k2   |
+------+------+
| NULL | B    |
| NULL | b    |
| A    | NULL |
| A    | B    |
| A    | b    |
| a    | NULL |
| a    | B    |
| a    | b    |
+------+------+
8 rows in set (0.12 sec)

MySQL > SELECT k1, k2, GROUPING(k1), GROUPING(k2), COUNT(*) FROM t GROUP BY ROLLUP(k1, k2);
+------+------+--------------+--------------+----------+
| k1   | k2   | grouping(k1) | grouping(k2) | count(*) |
+------+------+--------------+--------------+----------+
| NULL | NULL |            1 |            1 |        8 |
| NULL | NULL |            0 |            1 |        2 |
| NULL | B    |            0 |            0 |        1 |
| NULL | b    |            0 |            0 |        1 |
| A    | NULL |            0 |            1 |        3 |
| a    | NULL |            0 |            1 |        3 |
| A    | NULL |            0 |            0 |        1 |
| A    | B    |            0 |            0 |        1 |
| A    | b    |            0 |            0 |        1 |
| a    | NULL |            0 |            0 |        1 |
| a    | B    |            0 |            0 |        1 |
| a    | b    |            0 |            0 |        1 |
+------+------+--------------+--------------+----------+
12 rows in set (0.12 sec)
```