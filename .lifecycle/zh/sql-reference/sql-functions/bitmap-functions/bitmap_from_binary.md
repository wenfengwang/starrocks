---
displayed_sidebar: English
---

# 从二进制到位图

## 描述

将特定格式的二进制字符串转换成位图。

这个函数可以用来将位图数据加载到StarRocks中。

该函数从v3.0版本开始支持。

## 语法

```Haskell
BITMAP bitmap_from_binary(VARBINARY str)
```

## 参数

str：支持的数据类型为VARBINARY。

## 返回值

返回一个BITMAP类型的值。

## 示例

示例1：结合其他位图函数使用此函数。

```Plain
mysql> select bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string("0,1,2,3"))));
+---------------------------------------------------------------------------------------+
| bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string('0,1,2,3')))) |
+---------------------------------------------------------------------------------------+
| 0,1,2,3                                                                               |
+---------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```
