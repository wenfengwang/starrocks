---
displayed_sidebar: English
---

# var_samp, variance_samp

## 描述

返回表达式的样本方差。从 v2.5.10 版本开始，此函数还可以用作窗口函数。

## 语法

```Haskell
VAR_SAMP(expr)
```

## 参数

`expr`：要计算的表达式。如果它是表列，则其计算结果必须为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

## 返回值

返回一个 DOUBLE 值。

## 例子

```plaintext
MySQL > select var_samp(scan_rows)
from log_statis
group by datetime;
+-----------------------+
| var_samp(`scan_rows`) |
+-----------------------+
|    5.6227132145741789 |
+-----------------------+
```

## 关键词

VAR_SAMP, VARIANCE_SAMP, VAR, SAMP, 方差
