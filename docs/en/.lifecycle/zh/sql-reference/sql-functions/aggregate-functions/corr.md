---
displayed_sidebar: English
---

# corr

## 描述

返回两个表达式之间的皮尔逊相关系数。该函数从 v2.5.10 版本开始支持，并且可以作为窗口函数使用。

## 语法

```Haskell
CORR(expr1, expr2)
```

## 参数

`expr1` 和 `expr2` 必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL 类型。

如果 `expr1` 和 `expr2` 是表的列，则该函数计算这两列的相关系数。

## 返回值

返回 DOUBLE 类型的值。公式如下，其中 `n` 代表表的行数：

![corr formula](../../../assets/corr_formula.png)

<!--$$
\frac{\sum_{i=1}^{n}((x_i - \bar{x})(y_i - \bar{y}))}{\sqrt{\sum_{i=1}^{n}((x_i - \bar{x})^2) \cdot \sum_{i=1}^{n}((y_i - \bar{y})^2)}}

## 使用说明

- 只有当数据行中的两列都是非空值时，该数据行才会被计入统计。否则，这条数据行会被排除在结果之外。

- 如果 `n` 为 1，则返回 0。

- 如果任一输入为 NULL，则返回 NULL。

## 示例

假设表 `agg` 有以下数据：

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

计算 `k` 和 `v` 列的相关系数：

```plaintext
mysql> select no,CORR(k,v) from agg group by no;
+------+--------------------+
| no   | corr(k, v)         |
+------+--------------------+
|    1 |               NULL |
|    2 | 0.9988445981121532 |
+------+--------------------+
```