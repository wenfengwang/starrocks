---
displayed_sidebar: English
---

# 空格

## 描述

返回一个由指定数量空格组成的字符串。

## 语法

```Haskell
space(x);
```

## 参数

x：要返回的空格数量。支持的数据类型为 INT。

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
mysql> select space(6);
+----------+
| space(6) |
+----------+
|          |
+----------+
1 row in set (0.00 sec)
```

## 关键字

SPACE
