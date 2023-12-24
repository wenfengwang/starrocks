---
displayed_sidebar: English
---

# covar_samp

## 描述

返回两个表达式的样本协方差。从 v2.5.10 开始支持此函数。它也可以作为窗口函数使用。

## 语法

```Haskell
COVAR_SAMP(expr1, expr2)
```

## 参数

`expr1` 和 `expr2` 必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

如果 `expr1` 和 `expr2` 是表列，则此函数计算这两列的样本协方差。

## 返回值

返回一个 DOUBLE 值。公式如下，其中 `n` 表示表的行数:

![covar_samp公式](../../../assets/covar_samp_formula.png)

<!--$$
\frac{\sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})}{n-1}
$$-->

## 使用说明

- 仅当该行中的两列为非 null 值时，才会对该数据行进行计数。否则，将从结果中删除此数据行。

- 如果 `n` 为 1，则返回 0。

- 如果任一输入为 NULL，则返回 NULL。

## 例子

假设表 `agg` 包含以下数据：

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

计算 `k` 和 `v` 列的样本协方差：

```plaintext
mysql> select no,COVAR_SAMP(k,v) from agg group by no;
+------+--------------------+
| no   | covar_samp(k, v)   |
+------+--------------------+
|    1 |               NULL |
|    2 | 119.99999999999999 |
+------+--------------------+