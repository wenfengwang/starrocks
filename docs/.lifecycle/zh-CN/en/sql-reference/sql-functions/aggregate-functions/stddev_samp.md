---
displayed_sidebar: "Chinese"
---

# stddev_samp

## 描述

返回表达式的样本标准差。自v2.5.10起，此函数也可以用作窗口函数。

## 语法

```Haskell
STDDEV_SAMP(expr)
```

## 参数

`expr`: 表达式。如果它是表列，则必须计算为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

## 返回值

返回DOUBLE值。

## 示例

```plain text
MySQL > select stddev_samp(scan_rows)
from log_statis
group by datetime;
+--------------------------+
| stddev_samp(`scan_rows`) |
+--------------------------+
|        2.372044195280762 |
+--------------------------+
```

## 关键词

STDDEV_SAMP,STDDEV,SAMP