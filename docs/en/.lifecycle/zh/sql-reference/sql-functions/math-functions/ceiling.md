---
displayed_sidebar: English
---

# 天花板

## 描述

返回输入参数`arg`四舍五入后最接近或更大的整数值。

## 语法

```Shell
ceiling(arg)
```

## 参数

`arg` 支持 DOUBLE 数据类型。

## 返回值

返回一个 BIGINT 数据类型的值。

## 例子

```Plain
mysql> select ceiling(3.14);
+---------------+
| ceiling(3.14) |
+---------------+
|             4 |
+---------------+
1 行受影响 (0.00 秒)