---
displayed_sidebar: English
---

# 标准差_样本

## 描述

计算表达式的样本标准差。自v2.5.10版本起，此函数亦可作为窗口函数使用。

## 语法

```Haskell
STDDEV_SAMP(expr)
```

## 参数

expr：表达式。若为表格列，则其结果须为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL 类型。

## 返回值

返回一个 DOUBLE 类型的值。

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

STDDEV_SAMP、STDDEV、SAMP
