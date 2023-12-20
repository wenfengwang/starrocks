---
displayed_sidebar: English
---

# 以...开头

## 描述

当一个字符串以特定的前缀开头时，该函数返回1；如果不是，则返回0。当参数为NULL时，结果也是NULL。

## 语法

```Haskell
BOOLEAN starts_with(VARCHAR str, VARCHAR prefix)
```

## 示例

```Plain
mysql> select starts_with("hello world","hello");
+-------------------------------------+
|starts_with('hello world', 'hello')  |
+-------------------------------------+
| 1                                   |
+-------------------------------------+

mysql> select starts_with("hello world","world");
+-------------------------------------+
|starts_with('hello world', 'world')  |
+-------------------------------------+
| 0                                   |
+-------------------------------------+
```

## 关键字

STARTS_WITH
