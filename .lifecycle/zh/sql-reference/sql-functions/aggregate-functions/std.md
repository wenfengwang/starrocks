---
displayed_sidebar: English
---

# 标准差

## 描述

返回表达式的标准差。从v2.5.10版本开始，此函数还可以作为窗口函数使用。

## 语法

```Haskell
STD(expr)
```

## 参数

expr：表达式。如果是表格列，则其计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL 类型。

## 返回值

返回一个 DOUBLE 类型的值。

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

计算 col0 和 col1 的标准差。

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
