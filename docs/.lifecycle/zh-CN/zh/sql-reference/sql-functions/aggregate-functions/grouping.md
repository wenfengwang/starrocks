---
displayed_sidebar: "中文"
---

# 分组

## 功能

判斷一個列是否為聚合列，如果是聚合列則返回 0，否則返回 1。

## 語法

```Haskell
GROUPING(col_expr)
```

## 參數說明

`col_expr`: GROUP BY 子句的 ROLLUP、CUBE 或 GROUPING SETS 擴展的表達式列表中指定的維度列的表達式。

## 返回值說明

返回值為數值類型。

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