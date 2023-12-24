---
displayed_sidebar: English
---

# percentile_cont

## 描述

使用线性插值计算`expr`的百分位值。

## 语法

```Haskell
PERCENTILE_CONT (expr, percentile) 
```

## 参数

- `expr`：用于对值进行排序的表达式。它必须是数值数据类型、DATE 或 DATETIME。例如，如果要查找物理分数的中位数，请指定包含物理分数的列。

- `percentile`：要查找的值的百分位数。它是从 0 到 1 的常数浮点数。例如，如果要查找中值，请将此参数设置为 `0.5`。

## 返回值

返回指定百分位数处的值。如果没有输入值正好位于所需的百分位数，则使用两个最接近的输入值的线性插值来计算结果。

数据类型与 `expr` 相同。

## 使用说明

此函数忽略 NULL 值。

## 例子

假设有一张名为 `exam` 的表，其中包含以下数据。

```Plain
select * from exam order by Subject;
+-----------+-------+
| Subject   | Score |
+-----------+-------+
| chemistry |    80 |
| chemistry |   100 |
| chemistry |  NULL |
| math      |    60 |
| math      |    70 |
| math      |    85 |
| physics   |    75 |
| physics   |    80 |
| physics   |    85 |
| physics   |    99 |
+-----------+-------+
```

计算每个科目的中位数分数，同时忽略 NULL 值。

查询：

```SQL
SELECT Subject, PERCENTILE_CONT (Score, 0.5)  FROM exam group by Subject;
```

结果：

```Plain
+-----------+-----------------------------+
| Subject   | percentile_cont(Score, 0.5) |
+-----------+-----------------------------+
| chemistry |                          90 |
| math      |                          70 |
| physics   |                        82.5 |
+-----------+-----------------------------+
```