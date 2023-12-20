---
displayed_sidebar: English
---

# 二进制转换

## 描述

将输入参数 arg 转换为二进制形式。

## 语法

```Shell
bin(arg)
```

## 参数

arg：需要转换为二进制的输入值。该参数支持 BIGINT 数据类型。

## 返回值

返回的值为 VARCHAR 数据类型。

## 示例

```Plain
mysql> select bin(3);
+--------+
| bin(3) |
+--------+
| 11     |
+--------+
1 row in set (0.02 sec)
```
