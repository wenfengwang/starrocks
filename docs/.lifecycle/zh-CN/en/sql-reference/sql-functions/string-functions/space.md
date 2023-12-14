---
displayed_sidebar: "Chinese"
---

# 空格

## 描述

返回指定数量的空格的字符串。

## 语法

```Haskell
space(x);
```

## 参数

`x`: 要返回的空格数量。支持的数据类型为INT。

## 返回值

返回VARCHAR类型的值。

## 示例

```Plain Text
mysql> select space(6);
+----------+
| space(6) |
+----------+
|          |
+----------+
1 row in set (0.00 sec)
```

## 关键词

空格