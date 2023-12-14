---
displayed_sidebar: "Chinese"
---


# stddev,stddev_pop,std

## 描述

返回expr表达式的总体标准差。自v2.5.10起，此函数也可以用作窗口函数。

## 语法

```Haskell
STDDEV(expr)
```

## 参数

`expr`: 表达式。如果是表列，则必须计算为TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE或DECIMAL。

## 返回值

返回DOUBLE值。

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

STDDEV,STDDEV_POP,POP