---
displayed_sidebar: English
---

# 标准偏差

## 描述

返回表达式的标准偏差。从 v2.5.10 开始，此函数也可以作为窗口函数使用。

## 语法

```Haskell
STD(expr)
```

## 参数

`expr`：要计算标准偏差的表达式。如果它是表列，则其计算结果必须为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

## 返回值

返回一个 DOUBLE 值。

## 例子

```plaintext
MySQL > select * from std_test;
+------+------+
| col0 | col1 |
+------+------+
|    0 |    0 |
|    1 |    2 |
|    2 |    4 |
|    3 |    6 |
|    4 |    8 |
+------+------+
```

计算 `col0` 和 `col1` 的标准偏差。

```plaintext
MySQL > select std(col0) as std_of_col0, std(col1) as std_of_col1 from std_test;
+--------------------+--------------------+
| std_of_col0        | std_of_col1        |
+--------------------+--------------------+
| 1.4142135623730951 | 2.8284271247461903 |
+--------------------+--------------------+
```

## 关键词

标准偏差
