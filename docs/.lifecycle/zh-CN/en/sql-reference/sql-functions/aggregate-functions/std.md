---
displayed_sidebar: "Chinese"
---

# 标准差

## 描述

返回表达式的标准差。自v2.5.10版以来，此函数也可以用作窗口函数。

## 语法

```Haskell
STD(expr)
```

## 参数

`expr`：表达式。如果它是表列，则必须计算为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

## 返回值

返回一个DOUBLE值。

## 示例

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

计算`col0`和`col1`的标准差。

```plaintext
MySQL > select std(col0) as std_of_col0, std(col1) as std_of_col1 from std_test;
+--------------------+--------------------+
| std_of_col0        | std_of_col1        |
+--------------------+--------------------+
| 1.4142135623730951 | 2.8284271247461903 |
+--------------------+--------------------+
```

## 关键词

STD