---
displayed_sidebar: English
---


# 标准差，stddev_pop，std

## 描述

返回表达式 expr 的总体标准差。从 v2.5.10 开始，此函数也可用作窗口函数。

## 语法

```Haskell
STDDEV(expr)
```

## 参数

`expr`：表达式。如果它是表列，则其计算结果必须为 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL。

## 返回值

返回一个 DOUBLE 值。

## 例子

```plaintext
mysql> SELECT stddev(lo_quantity), stddev_pop(lo_quantity) from lineorder;
+---------------------+-------------------------+
| stddev(lo_quantity) | stddev_pop(lo_quantity) |
+---------------------+-------------------------+
|   14.43100708360797 |       14.43100708360797 |
+---------------------+-------------------------+
```

## 关键词

STDDEV，STDDEV_POP，POP
