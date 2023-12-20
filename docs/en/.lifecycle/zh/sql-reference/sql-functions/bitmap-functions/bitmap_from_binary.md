---
displayed_sidebar: English
---

# bitmap_from_binary

## 描述

将具有特定格式的二进制字符串转换为位图。

此函数可用于加载位图数据到StarRocks。

该函数自v3.0版本起支持。

## 语法

```Haskell
BITMAP bitmap_from_binary(VARBINARY str)
```

## 参数

`str`：支持的数据类型为VARBINARY。

## 返回值

返回BITMAP类型的值。

## 示例

示例 1：将此函数与其他Bitmap函数一起使用。

```Plain
mysql> select bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string("0,1,2,3"))));
+---------------------------------------------------------------------------------------+
| bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string('0,1,2,3')))) |
+---------------------------------------------------------------------------------------+
| 0,1,2,3                                                                               |
+---------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```