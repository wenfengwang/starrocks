---
displayed_sidebar: English
---

# degrees

## 描述

将角度 x 从弧度转换为度。

## 语法

```SQL
DEGREES(x);
```

## 参数

`x`：角度，以弧度表示。支持 DOUBLE 类型。

## 返回值

返回 DOUBLE 数据类型的值。

## 示例

```Plaintext
mysql> select degrees(3.1415926535898);
+--------------------------+
| degrees(3.1415926535898) |
+--------------------------+
|        180.0000000000004 |
+--------------------------+
1 row in set (0.07 sec)
```