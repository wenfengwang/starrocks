---
displayed_sidebar: English
---

# 方差, var_pop, variance_pop

## 描述

返回表达式的总体方差。从 v2.5.10 版本开始，此函数也可以作为窗口函数使用。

## 语法

```Haskell
VARIANCE(expr)
```

## 参数

`expr`：表达式。如果是表格列，其计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL 类型。

## 返回值

返回 DOUBLE 类型的值。

## 示例

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

VARIANCE, VAR_POP, VARIANCE_POP