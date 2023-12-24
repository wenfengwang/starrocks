---
displayed_sidebar: English
---

# CORR

## 描述

返回两个表达式之间的 Pearson 相关系数。从 v2.5.10 开始支持此功能。它也可以用作窗口函数。

## 语法

```Haskell
CORR(expr1, expr2)
```

## 参数

`expr1` 和 `expr2` 必须计算为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

如果 `expr1` 和 `expr2` 是表列，则此函数计算这两列的相关系数。

## 返回值

返回一个 DOUBLE 值。公式如下，其中 `n` 表示表的行数：

![corr 公式](../../../assets/corr_formula.png)

<!--$$
\frac{\sum_{i=1}^{n}((x_i - \bar{x})(y_i - \bar{y}))}{\sqrt{\sum_{i=1}^{n}((x_i - \bar{x})^2) \cdot \sum_{i=1}^{n}((y_i - \bar{y})^2)}}
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

计算 `k` 列和 `v` 列的相关系数：

```plaintext
mysql> select no,CORR(k,v) from agg group by no;
+------+--------------------+
| no   | corr(k, v)         |
+------+--------------------+
|    1 |               NULL |
|    2 | 0.9988445981121532 |
+------+--------------------+
```
