---
displayed_sidebar: English
---

# 符号函数

## 说明

返回数值 x 的符号。输入为负数、0 或正数时，分别输出 -1、0 或 1。

## 语法

```Haskell
SIGN(x);
```

## 参数

x：支持 DOUBLE 数据类型。

## 返回值

返回的是 FLOAT 数据类型的值。

## 示例

```Plain
mysql> select sign(3.14159);
+---------------+
| sign(3.14159) |
+---------------+
|             1 |
+---------------+
1 row in set (0.02 sec)
```
