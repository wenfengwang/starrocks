---
displayed_sidebar: "Chinese"
---

# bitmap_from_binary

## Description

将具有特定格式的二进制字符串转换为位图。

此函数可用于将位图数据加载到 StarRocks。

此函数从v3.0开始支持。

## Syntax

```Haskell
BITMAP bitmap_from_binary(VARBINARY str)
```

## Parameters

`str`：支持的数据类型为VARBINARY。

## Return value

返回BITMAP类型的值。

## Examples

示例1：将此函数与其他位图函数一起使用。

```Plain
mysql> select bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string("0,1,2,3"))));
+---------------------------------------------------------------------------------------+
| bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string('0,1,2,3')))) |
+---------------------------------------------------------------------------------------+
| 0,1,2,3                                                                               |
+---------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```