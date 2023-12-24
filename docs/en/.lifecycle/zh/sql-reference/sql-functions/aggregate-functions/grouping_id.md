---
displayed_sidebar: English
---


# grouping_id

## 描述

grouping_id 用于区分相同分组标准的分组统计结果。

## 语法

```Haskell
GROUPING_ID(expr)
```

## 例子

```Plain
MySQL > SELECT COL1, GROUPING_ID(COL2) AS 'GroupingID' FROM tbl GROUP BY ROLLUP (COL1, COL2);
+------+------------+
| COL1 | GroupingID |
+------+------------+
| NULL |          1 |
| 2.20 |          1 |
| 2.20 |          0 |
| 1.10 |          1 |
| 1.10 |          0 |
+------+------------+