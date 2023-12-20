---
displayed_sidebar: English
---

# 百分位数连续

## 描述

计算表达式 expr 的百分位数值，并采用线性插值法。

## 语法

```Haskell
PERCENTILE_CONT (expr, percentile) 
```

## 参数

- expr：用于对值进行排序的表达式。它必须是数值型数据类型、日期（DATE）或日期时间（DATETIME）。例如，如果您想要找出物理成绩的中位数，请指定包含物理成绩的列。

- percentile：您想要查找的值的百分位数。这是一个从 0 到 1 的常数浮点数。例如，如果您想要找出中位数，那么将此参数设置为 0.5。

## 返回值

返回位于指定百分位的值。如果没有任何输入值正好位于期望的百分位上，那么结果将通过对最接近的两个输入值进行线性插值来计算得出。

返回值的数据类型与 expr 相同。

## 使用须知

此函数会忽略 NULL 值。

## 示例

假设存在一个名为 exam 的表格，其中含有以下数据。

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

计算忽略 NULL 值的情况下，每门科目的中位成绩。

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
