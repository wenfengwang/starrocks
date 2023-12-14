---
displayed_sidebar: "English"
---

# ceil, dceil

## 描述

返回将输入的参数 `arg` 四舍五入到最接近的等于或大于它的整数值。

## 语法

```Shell
ceil(arg)
```

## 参数

`arg` 支持 DOUBLE 数据类型。

## 返回值

返回一个 BIGINT 数据类型的值。

## 示例

```Plain
mysql> select ceil(3.14);
+------------+
| ceil(3.14) |
+------------+
|          4 |
+------------+
1 行受影响 (0.15 秒)
```