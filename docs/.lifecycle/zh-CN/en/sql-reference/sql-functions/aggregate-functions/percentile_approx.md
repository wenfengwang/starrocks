---
displayed_sidebar: "Chinese"
---

# 百分位数估计

## 描述

返回第p百分位数的估计值，其中p的值介于0和1之间。

压缩参数是可选的，设置范围为[2048, 10000]。值越大，精度越高，内存消耗越大，计算时间越长。如果未指定或不在[2048, 10000]范围内，则函数将使用默认的压缩参数10000运行。

此函数使用固定大小的内存，因此可以在具有高基数的列上使用更少的内存，并可用于计算诸如tp99之类的统计信息。

## 语法

```Haskell
PERCENTILE_APPROX(expr, DOUBLE p[, DOUBLE compression])
```

## 示例

```plain text
MySQL > select `table`, percentile_approx(cost_time,0.99)
from log_statis
group by `table`;
+----------+--------------------------------------+
| table    | percentile_approx(`cost_time`, 0.99) |
+----------+--------------------------------------+
| test     |                                54.22 |
+----------+--------------------------------------+

MySQL > select `table`, percentile_approx(cost_time,0.99, 4096)
from log_statis
group by `table`;
+----------+----------------------------------------------+
| table    | percentile_approx(`cost_time`, 0.99, 4096.0) |
+----------+----------------------------------------------+
| test     |                                        54.21 |
+----------+----------------------------------------------+
```

## 关键词

PERCENTILE_APPROX,PERCENTILE,APPROX