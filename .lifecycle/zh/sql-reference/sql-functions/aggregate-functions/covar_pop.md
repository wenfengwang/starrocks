---
displayed_sidebar: English
---

# 总体协方差函数

## 描述

返回两个表达式的总体协方差。该函数从 v2.5.10 版本开始支持，并且可以作为窗口函数使用。

## 语法

```Haskell
COVAR_POP(expr1, expr2)
```

## 参数

expr1 和 expr2 必须计算结果为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL 类型。

如果 expr1 和 expr2 是表中的列，则该函数计算这两个列的总体协方差。

## 返回值

返回一个 DOUBLE 类型的值。公式如下，其中 n 代表表中的行数：

![covar_pop formula](../../../assets/covar_pop_formula.png)

<!--$$
\frac{\sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})}{n}

## 使用注意事项

- 只有当数据行中的两个列值均非空时，该行数据才会被计入计数。否则，这行数据将从结果中排除。

- 如果任何输入值为 NULL，则返回 NULL。

## 示例

假设表 agg 包含以下数据：

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

计算 k 列和 v 列的总体协方差：

```plaintext
mysql> select no,COVAR_POP(k,v) from agg group by no;
+------+-------------------+
| no   | covar_pop(k, v)   |
+------+-------------------+
|    1 |              NULL |
|    2 | 79.99999999999999 |
+------+-------------------+
```
