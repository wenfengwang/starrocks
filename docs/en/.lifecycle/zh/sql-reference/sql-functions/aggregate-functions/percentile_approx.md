---
displayed_sidebar: English
---

# percentile_approx

## 描述

返回第 p 个百分位的近似值，其中 p 的值介于 0 和 1 之间。

压缩参数是可选的，设置范围为 [2048, 10000]。值越大，精度越高，内存消耗越大，计算时间越长。如果未指定或不在 [2048, 10000] 范围内，则函数以默认压缩参数 10000 运行。

该函数使用固定大小的内存，因此对于基数较高的列可以使用较少的内存，并且可以用于计算如 tp99 等统计数据。

## 语法

```Haskell
PERCENTILE_APPROX(expr, DOUBLE p[, DOUBLE compression])
```

## 示例

```plain
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

## 关键字

PERCENTILE_APPROX, PERCENTILE, APPROX