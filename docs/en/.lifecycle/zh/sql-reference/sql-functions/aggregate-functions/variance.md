---
displayed_sidebar: English
---

# 方差，VAR_POP，VARIANCE_POP

## 描述

返回表达式的总体方差。从 v2.5.10 开始，此函数还可用作窗口函数。

## 语法

```Haskell
VARIANCE(expr)
```

## 参数

`expr`：要计算方差的表达式。如果它是表列，则其计算结果必须为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

## 返回值

返回一个 DOUBLE 值。

## 例子

```plaintext
MySQL > select var_pop(i_current_price), i_rec_start_date from item group by i_rec_start_date;
+--------------------------+------------------+
| var_pop(i_current_price) | i_rec_start_date |
+--------------------------+------------------+
|       314.96177792808226 | 1997-10-27       |
|       463.73633459357285 | NULL             |
|       302.02102643609123 | 1999-10-28       |
|        337.9318386924913 | 2000-10-27       |
|       333.80931439318346 | 2001-10-27       |
+--------------------------+------------------+

MySQL > select variance(i_current_price), i_rec_start_date from item group by i_rec_start_date;
+---------------------------+------------------+
| variance(i_current_price) | i_rec_start_date |
+---------------------------+------------------+
|        314.96177792808226 | 1997-10-27       |
|         463.7363345935729 | NULL             |
|        302.02102643609123 | 1999-10-28       |
|         337.9318386924912 | 2000-10-27       |
|        333.80931439318346 | 2001-10-27       |
+---------------------------+------------------+
```

## 关键词

方差，VAR_POP，VARIANCE_POP