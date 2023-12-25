---
displayed_sidebar: Chinese
---

# grouping

## 機能

ある列が集約列かどうかを判断し、集約列であれば 0 を、そうでなければ 1 を返します。

## 文法

```Haskell
GROUPING(col_expr)
```

## 引数説明

`col_expr`: GROUP BY 句の ROLLUP、CUBE、または GROUPING SETS の拡張で指定された次元列の式リストにある式。

## 戻り値の説明

戻り値は数値型です。

## 例

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
| k1   | k2   | GROUPING(k1) | GROUPING(k2) | COUNT(*) |
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
