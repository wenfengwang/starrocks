---
displayed_sidebar: English
---

# percentile_cont

## 描述

使用线性插值计算 `expr` 的百分位数值。

## 语法

```Haskell
PERCENTILE_CONT (expr, percentile) 
```

## 参数

- `expr`：用于对值进行排序的表达式。它必须是数值数据类型、DATE 或 DATETIME。例如，如果你想找出物理成绩的中位数，请指定包含物理成绩的列。

- `percentile`：你想要找到的值的百分位。它是一个从 0 到 1 的常数浮点数。例如，如果你想找到中位数，将此参数设置为 `0.5`。

## 返回值

返回位于指定百分位的值。如果没有输入值恰好位于所需的百分位，则结果通过两个最接近的输入值的线性插值计算得出。

数据类型与 `expr` 相同。

## 使用说明

此函数忽略 NULL 值。

## 示例

假设有一个名为 `exam` 的表，包含以下数据。

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

计算每个科目忽略 NULL 值的中位数分数。

查询：

```SQL
SELECT Subject, PERCENTILE_CONT (Score, 0.5) FROM exam GROUP BY Subject;
```

结果：

```Plain
+-----------+-----------------------------+
| Subject   | PERCENTILE_CONT(Score, 0.5) |
+-----------+-----------------------------+
| chemistry |                          90 |
| math      |                          70 |
| physics   |                        82.5 |
+-----------+-----------------------------+
```