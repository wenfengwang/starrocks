---
displayed_sidebar: English
---

# space

## 描述

返回指定数量的空格字符串。

## 语法

```Haskell
space(x);
```

## 参数

`x`：要返回的空格数量。支持的数据类型是 INT。

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
1 行在集合中 (0.00 秒)
```

## 关键词

SPACE