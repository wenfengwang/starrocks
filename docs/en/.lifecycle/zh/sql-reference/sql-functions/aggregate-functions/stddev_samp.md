---
displayed_sidebar: English
---

# stddev_samp

## 描述

返回表达式的样本标准偏差。自v2.5.10版本起，此函数也可作为窗口函数使用。

## 语法

```Haskell
STDDEV_SAMP(expr)
```

## 参数

`expr`：表达式。如果是表格列，其计算结果必须为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL 类型。

## 返回值

返回 DOUBLE 类型的值。

## 示例

```plain
MySQL > select stddev_samp(scan_rows)
from log_statis
group by datetime;
+--------------------------+
| stddev_samp(`scan_rows`) |
+--------------------------+
|        2.372044195280762 |
+--------------------------+
```

## 关键字

STDDEV_SAMP,STDDEV,SAMP