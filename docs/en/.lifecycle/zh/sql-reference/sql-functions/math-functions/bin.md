---
displayed_sidebar: English
---

# bin

## 描述

将输入的 `arg` 转换为二进制形式。

## 语法

```Shell
bin(arg)
```

## 参数

`arg`：要转换为二进制的输入。支持 BIGINT 数据类型。

## 返回值

返回 VARCHAR 数据类型的值。

## 例子

```Plain
mysql> select bin(3);
+--------+
| bin(3) |
+--------+
| 11     |
+--------+
1 行记录(0.02 秒)