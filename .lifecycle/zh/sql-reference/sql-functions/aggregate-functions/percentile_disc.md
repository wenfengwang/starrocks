---
displayed_sidebar: English
---

# 百分位数_离散

## 描述

根据输入列 expr 的离散分布返回百分位数值。如果无法找到精确的百分位数值，该函数将返回两个最接近值中的较大者。

该功能从 v2.5 版本开始提供支持。

## 语法

```SQL
PERCENTILE_DISC (expr, percentile) 
```

## 参数

- expr：您要计算百分位数的列。该列可以是任意可排序的数据类型。
- 百分位数：您希望找到的值的百分位。它必须是介于 0 和 1 之间的常数浮点数。例如，如果您想要找到中位数，将此参数设置为 0.5。如果您想找到第 70 百分位的值，指定 0.7。

## 返回值

返回值的数据类型与 expr 的数据类型相同。

## 使用说明

在计算时会忽略 NULL 值。

## 示例

创建表 exam 并向该表中插入数据。

```sql
CREATE TABLE exam (
    subject STRING,
    exam_result INT
) 
DISTRIBUTED BY HASH(`subject`);

insert into exam values
('chemistry',80),
('chemistry',100),
('chemistry',null),
('math',60),
('math',70),
('math',85),
('physics',75),
('physics',80),
('physics',85),
('physics',99);
```

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

计算每个科目的中位数。

```SQL
SELECT Subject, PERCENTILE_DISC (Score, 0.5)
FROM exam group by Subject;
```

输出结果

```Plain
+-----------+-----------------------------+
| Subject   | percentile_disc(Score, 0.5) |
+-----------+-----------------------------+
| chemistry |                         100 |
| math      |                          70 |
| physics   |                          85 |
+-----------+-----------------------------+
```
