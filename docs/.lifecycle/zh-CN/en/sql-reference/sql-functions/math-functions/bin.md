---
displayed_sidebar: "Chinese"
---

# bin

## 描述

将输入的`arg`转换为二进制。

## 语法

```Shell
bin(arg)
```

## 参数

`arg`：要转换为二进制的输入。支持BIGINT数据类型。

## 返回值

返回VARCHAR数据类型的值。

## 示例

```Plain
mysql> select bin(3);
+--------+
| bin(3) |
+--------+
| 11     |
+--------+
1行记录受影响（用时0.02秒）
```