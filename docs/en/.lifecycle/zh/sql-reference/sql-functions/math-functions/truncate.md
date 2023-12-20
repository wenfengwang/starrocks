---
displayed_sidebar: English
---

# 截断

## 描述

将输入值向下舍入到最接近的等于或小于指定小数点后位数的值。

## 语法

```Shell
truncate(arg1,arg2);
```

## 参数

- `arg1`：要舍入的输入值。支持以下数据类型：
  - DOUBLE
  - DECIMAL128
- `arg2`：小数点后要保留的位数。支持 INT 数据类型。

## 返回值

返回一个与 `arg1` 相同数据类型的值。

## 示例

```Plain
mysql> select truncate(3.14,1);
+-------------------+
| truncate(3.14, 1) |
+-------------------+
|               3.1 |
+-------------------+
1 row in set (0.00 sec)
```