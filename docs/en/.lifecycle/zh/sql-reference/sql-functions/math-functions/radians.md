---
displayed_sidebar: English
---

# 弧度

## 描述

将 `x` 从角度转换为弧度。

## 语法

```Haskell
RADIANS(x);
```

## 参数

`x`：支持 DOUBLE 数据类型。

## 返回值

返回一个 DOUBLE 数据类型的值。

## 例子

```Plain
mysql> select radians(90);
+--------------------+
| radians(90)        |
+--------------------+
| 1.5707963267948966 |
+--------------------+
1 行受影响 (0.00 秒)