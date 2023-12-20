---
displayed_sidebar: English
---

# 样本协方差_samp

## 描述

返回两个表达式的样本协方差。该函数从 v2.5.10 版本开始支持，并且可作为窗口函数使用。

## 语法

```Haskell
COVAR_SAMP(expr1, expr2)
```

## 参数

expr1 和 expr2 必须计算为以下类型之一：TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

如果 expr1 和 expr2 是表中的列，则该函数计算这两个列的样本协方差。

## 返回值

返回一个 DOUBLE 类型的值。其公式如下，其中 n 代表表中的行数：

![covar_samp formula](../../../assets/covar_samp_formula.png)

<!--$$
\frac{\sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})}{n-1}

## 使用说明

- 只有当数据行中的两个列值均非空时，该数据行才会被计入计数。否则，该数据行将被排除在结果之外。

- 如果 n 等于 1，则返回值为 0。

- 如果任一输入为 NULL，则返回 NULL。

## 示例

假设表 agg 包含如下数据：

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

计算 k 列和 v 列的样本协方差：

```plaintext
mysql> select no,COVAR_SAMP(k,v) from agg group by no;
+------+--------------------+
| no   | covar_samp(k, v)   |
+------+--------------------+
|    1 |               NULL |
|    2 | 119.99999999999999 |
+------+--------------------+
```
