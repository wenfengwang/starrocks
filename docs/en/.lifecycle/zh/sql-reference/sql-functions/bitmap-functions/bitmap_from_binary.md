---
displayed_sidebar: English
---

# bitmap_from_binary

## 描述

将特定格式的二进制字符串转换为位图。

此函数可用于将位图数据加载到 StarRocks。

该函数从 v3.0 版本开始支持。

## 语法

```Haskell
BITMAP bitmap_from_binary(VARBINARY str)
```

## 参数

`str`：支持的数据类型为 VARBINARY。

## 返回值

返回一个 BITMAP 类型的值。

## 例子

示例 1：将此函数与其他位图函数一起使用。

```Plain
mysql> select bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string("0,1,2,3"))));
+---------------------------------------------------------------------------------------+
| bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string('0,1,2,3')))) |
+---------------------------------------------------------------------------------------+
| 0,1,2,3                                                                               |
+---------------------------------------------------------------------------------------+
1 行记录 (0.01 秒)