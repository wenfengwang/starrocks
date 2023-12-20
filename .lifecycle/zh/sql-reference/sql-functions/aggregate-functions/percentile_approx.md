---
displayed_sidebar: English
---

# 百分位数近似

## 描述

返回 p 百分位数的近似值，其中 p 的取值范围在 0 到 1 之间。

压缩参数是可选的，其设置范围为 [2048, 10000]。参数值越大，精度越高，相应的内存消耗和计算时间也会增加。如果未指定该参数或其值未超出 [2048, 10000] 的范围，则函数将采用默认的压缩参数 10000 运行。

此函数使用固定大小的内存，因此对于基数较高的列可以减少内存使用，并且能够用于计算如 tp99 等统计指标。

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
