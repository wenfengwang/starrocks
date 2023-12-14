---
displayed_sidebar: "Chinese"
---

# percentile_disc

## 描述

根据输入列`expr`的离散分布返回基于百分位的值。如果无法找到确切的百分位值，此函数将返回最接近的两个值中较大的一个。

此函数从v2.5版本开始受支持。

## 语法

```SQL
PERCENTILE_DISC (expr, percentile) 
```

## 参数

- `expr`: 您要计算百分位值的列。该列可以是任何可排序的数据类型。
- `percentile`: 您要查找其值的百分位数。必须是0和1之间的常数浮点数。例如，如果要查找中位数，将此参数设置为`0.5`。如果要查找70百分位数的值，请指定`0.7`。

## 返回值

返回值的数据类型与`expr`相同。

## 使用说明

在计算中忽略NULL值。

## 示例

创建表`exam`并将数据插入该表。

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

输出

```Plain
+-----------+-----------------------------+
| Subject   | percentile_disc(Score, 0.5) |
+-----------+-----------------------------+
| chemistry |                         100 |
| math      |                          70 |
| physics   |                          85 |
+-----------+-----------------------------+
```