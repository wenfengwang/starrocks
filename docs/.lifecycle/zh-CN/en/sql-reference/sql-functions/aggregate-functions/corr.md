---
displayed_sidebar: "Chinese"
---

# CORR

## 描述

返回两个表达式之间的Pearson相关系数。 从v2.5.10开始支持此函数。 它也可以用作窗口函数。

## 语法

```Haskell
CORR(expr1, expr2)
```

## 参数

`expr1`和`expr2`必须求值为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

如果`expr1`和`expr2`是表列，则此函数计算这两列的相关系数。

## 返回值

返回一个DOUBLE值。 公式如下，其中`n`表示表的行数：

![corr formula](../../../assets/corr_formula.png)

<!--$$
\frac{\sum_{i=1}^{n}((x_i - \bar{x})(y_i - \bar{y}))}{\sqrt{\sum_{i=1}^{n}((x_i - \bar{x})^2) \cdot \sum_{i=1}^{n}((y_i - \bar{y})^2)}}
$$-->

## 使用注意事项

- 仅当此行中的两列为非空值时才计算数据行。 否则，此数据行将从结果中消除。

- 如果`n`为1，则返回0。

- 如果任何输入为NULL，则返回NULL。

## 示例

假设表`agg`包含以下数据：

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

计算`k`和`v`列的相关系数：

```plaintext
mysql> select no,CORR(k,v) from agg group by no;
+------+--------------------+
| no   | corr(k, v)         |
+------+--------------------+
|    1 |               NULL |
|    2 | 0.9988445981121532 |
+------+--------------------+
```