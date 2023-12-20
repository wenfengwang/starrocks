---
displayed_sidebar: English
---

# percentile_disc

## 描述

根据输入列 `expr` 的离散分布返回百分位数值。如果无法找到确切的百分位数值，此函数返回两个最接近值中的较大者。

该函数从 v2.5 版本开始支持。

## 语法

```SQL
PERCENTILE_DISC (expr, percentile) 
```

## 参数

- `expr`：你想要计算百分位数的列。该列可以是任何可排序的数据类型。
- `percentile`：你想要找到的值的百分位数。它必须是介于 0 和 1 之间的常数浮点数。例如，如果你想要找到中位数，应将此参数设置为 `0.5`。如果你想找到第 70 百分位的值，应指定 0.7。

## 返回值

返回值的数据类型与 `expr` 相同。

## 使用说明

在计算中会忽略 NULL 值。

## 示例

创建表 `exam` 并向该表中插入数据。

```sql
CREATE TABLE exam (
    subject STRING,
    exam_result INT
) 
DISTRIBUTED BY HASH(`subject`);

INSERT INTO exam VALUES
('chemistry',80),
('chemistry',100),
('chemistry',NULL),
('math',60),
('math',70),
('math',85),
('physics',75),
('physics',80),
('physics',85),
('physics',99);
```

```Plain
SELECT * FROM exam ORDER BY subject;
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
SELECT subject, PERCENTILE_DISC (exam_result, 0.5)
FROM exam GROUP BY subject;
```

输出

```Plain
+-----------+-----------------------------+
| Subject   | PERCENTILE_DISC(exam_result, 0.5) |
+-----------+-----------------------------+
| chemistry |                           100 |
| math      |                            70 |
| physics   |                            85 |
+-----------+-----------------------------+
```