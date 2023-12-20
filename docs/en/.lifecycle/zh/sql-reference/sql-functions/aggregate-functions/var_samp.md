---
displayed_sidebar: English
---

# var_samp,variance_samp

## 描述

返回表达式的样本方差。自v2.5.10起，此函数也可作为窗口函数使用。

## 语法

```Haskell
VAR_SAMP(expr)
```

## 参数

`expr`：表达式。如果是表格列，则其计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL 类型。

## 返回值

返回一个 DOUBLE 类型的值。

## 示例

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

## 关键字

VAR_SAMP、VARIANCE_SAMP、VAR、SAMP、VARIANCE