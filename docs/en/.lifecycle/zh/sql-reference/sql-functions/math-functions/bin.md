---
displayed_sidebar: English
---

# bin

## 描述

将输入参数 `arg` 转换为二进制形式。

## 语法

```Shell
bin(arg)
```

## 参数

`arg`：你想要转换成二进制的输入值。它支持 BIGINT 数据类型。

## 返回值

返回一个 VARCHAR 数据类型的值。

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