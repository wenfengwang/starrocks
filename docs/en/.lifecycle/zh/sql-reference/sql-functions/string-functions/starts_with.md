---
displayed_sidebar: English
---

# starts_with

## 描述

当字符串以指定前缀开头时，此函数返回 1。否则，它返回 0。当参数为 NULL 时，结果也是 NULL。

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

START_WITH