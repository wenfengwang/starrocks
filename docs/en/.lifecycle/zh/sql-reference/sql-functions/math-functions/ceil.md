---
displayed_sidebar: English
---

# ceil, dceil

## 描述

返回输入`arg`四舍五入到最接近的等于或更大的整数的值。

## 语法

```Shell
ceil(arg)
```

## 参数

`arg` 支持 DOUBLE 数据类型。

## 返回值

返回 BIGINT 数据类型的值。

## 例子

```Plain
mysql> select ceil(3.14);
+------------+
| ceil(3.14) |
+------------+
|          4 |
+------------+
1 行受影响 (0.15 秒)