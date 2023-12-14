---
displayed_sidebar: "Chinese"
---

# ceiling

## 描述

将输入的参数`arg`的值四舍五入为最接近的等于或大于的整数。

## 语法

```Shell
ceiling(arg)
```

## 参数

`arg` 支持DOUBLE数据类型。

## 返回值

返回BIGINT数据类型的值。

## 示例

```Plain
mysql> select ceiling(3.14);
+---------------+
| ceiling(3.14) |
+---------------+
|             4 |
+---------------+
1 行输入 (0.00 秒)
```