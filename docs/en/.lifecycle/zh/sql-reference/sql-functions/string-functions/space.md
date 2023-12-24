---
displayed_sidebar: English
---

# 空格

## 描述

返回指定数量的空格组成的字符串。

## 语法

```Haskell
space(x);
```

## 参数

`x`：要返回的空格数。支持的数据类型为 INT。

## 返回值

返回 VARCHAR 类型的值。

## 例子

```Plain Text
mysql> select space(6);
+----------+
| space(6) |
+----------+
|          |
+----------+
1 行受影响 (0.00 秒)
```

## 关键字

SPACE