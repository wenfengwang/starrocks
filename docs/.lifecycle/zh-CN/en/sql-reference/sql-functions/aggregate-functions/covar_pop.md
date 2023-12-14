---
displayed_sidebar: "Chinese"
---

# covar_pop

## 描述

返回两个表达式的总体协方差。此函数支持从v2.5.10开始。它也可以作为窗口函数使用。

## 语法

```Haskell
COVAR_POP(expr1, expr2)
```

## 参数

`expr1`和`expr2`必须评估为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

如果`expr1`和`expr2`是表列，则此函数计算这两列的总体协方差。

## 返回值

返回DOUBLE值。公式如下，其中`n`表示表的行数：

![covar_pop公式](../../../assets/covar_pop_formula.png)

<!--$$
\frac{\sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})}{n}
$$-->

## 使用说明

- 仅当此行中的两列是非空值时才计算数据行。否则，该数据行将从结果中删除。

- 如果任何输入为NULL，则返回NULL。

## 示例

假设表`agg`具有以下数据：

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

计算`k`和`v`列的总体协方差：

```plaintext
mysql> select no,COVAR_POP(k,v) from agg group by no;
+------+-------------------+
| no   | covar_pop(k, v)   |
+------+-------------------+
|    1 |              NULL |
|    2 | 79.99999999999999 |
+------+-------------------+
```