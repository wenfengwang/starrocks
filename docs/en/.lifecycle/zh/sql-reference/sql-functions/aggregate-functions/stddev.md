---
displayed_sidebar: English
---


# stddev、stddev_pop、std

## 描述

返回 expr 表达式的总体标准差。自 v2.5.10 起，此函数还可以作为窗口函数使用。

## 语法

```Haskell
STDDEV(expr)
```

## 参数

`expr`：表达式。如果它是表的列，则其计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL 类型。

## 返回值

返回一个 DOUBLE 类型的值。

## 示例

```plaintext
mysql> SELECT stddev(lo_quantity), stddev_pop(lo_quantity) from lineorder;
+---------------------+-------------------------+
| stddev(lo_quantity) | stddev_pop(lo_quantity) |
+---------------------+-------------------------+
|   14.43100708360797 |       14.43100708360797 |
+---------------------+-------------------------+
```

## 关键字

STDDEV、STDDEV_POP、POP