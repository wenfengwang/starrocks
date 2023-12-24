---
displayed_sidebar: English
---

# stddev_samp

## 描述

返回表达式的样本标准差。从 v2.5.10 开始，此函数也可以作为窗口函数使用。

## 语法

```Haskell
STDDEV_SAMP(expr)
```

## 参数

`expr`：表达式。如果它是表列，则其计算结果必须为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

## 返回值

返回一个 DOUBLE 值。

## 例子

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

STDDEV_SAMP，STDDEV，SAMP
