---
displayed_sidebar: "Chinese"
---

# var_samp,variance_samp

## Description

返回表达式的样本方差。自v2.5.10起，此函数也可以用作窗口函数。

## Syntax

```Haskell
VAR_SAMP(expr)
```

## Parameters

`expr`: 表达式。如果它是表列，则必须计算为TINYINT, SMALLINT, INT, BIGINT, LARGEINT, FLOAT, DOUBLE或DECIMAL。

## Return value

返回一个DOUBLE值。

## Examples

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

## keyword

VAR_SAMP,VARIANCE_SAMP,VAR,SAMP,VARIANCE