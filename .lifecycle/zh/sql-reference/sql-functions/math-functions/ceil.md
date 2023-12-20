---
displayed_sidebar: English
---

# ceil, dceil

## 描述

返回输入参数 arg 四舍五入后的最接近的相等或更大整数值。

## 语法

```Shell
ceil(arg)
```

## 参数

arg 支持 DOUBLE 数据类型。

## 返回值

返回的是 BIGINT 数据类型的值。

## 示例

```Plain
mysql> select ceil(3.14);
+------------+
| ceil(3.14) |
+------------+
|          4 |
+------------+
1 row in set (0.15 sec)
```
