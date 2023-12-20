---
displayed_sidebar: English
---

# 向上取整

## 描述

返回输入参数 arg 四舍五入后最接近的相等或更大整数的值。

## 语法

```Shell
ceiling(arg)
```

## 参数

arg 支持 DOUBLE 数据类型。

## 返回值

返回的是 BIGINT 数据类型的值。

## 示例

```Plain
mysql> select ceiling(3.14);
+---------------+
| ceiling(3.14) |
+---------------+
|             4 |
+---------------+
1 row in set (0.00 sec)
```
