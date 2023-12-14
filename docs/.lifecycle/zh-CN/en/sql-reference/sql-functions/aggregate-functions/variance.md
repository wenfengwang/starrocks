---
displayed_sidebar: "中文"
---

# 方差，var_pop，variance_pop

## 描述

返回表达式的总体方差。 自v2.5.10起，此函数也可以用作窗口函数。

## 语法

```Haskell
VARIANCE(expr)
```

## 参数

`expr`：表达式。如果它是表列，则必须评估为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

## 返回值

返回DOUBLE值。

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

VARIANCE,VAR_POP,VARIANCE_POP