---
displayed_sidebar: English
---


# stddev、stddev_pop、std

## 描述

返回表达式 expr 的总体标准差。从 v2.5.10 版本开始，此函数还可以作为窗口函数使用。

## 语法

```Haskell
STDDEV(expr)
```

## 参数

expr：表达式。如果它是一个表格列，那么它的计算结果必须是 TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 或 DECIMAL 类型。

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

## 关键词

STDDEV、STDDEV_POP、POP
