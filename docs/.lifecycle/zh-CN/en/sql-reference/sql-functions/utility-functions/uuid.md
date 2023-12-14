---
displayed_sidebar: "Chinese"
---

# UUID

## 描述

返回一个VARCHAR类型的随机UUID。对此函数的两次调用可以生成两个不同的数。UUID的长度为36个字符。它包含5个十六进制数，用aaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee格式的四个连字符连接起来。

## 语法

```Haskell
uuid();
```

## 参数

无

## 返回值

返回一个VARCHAR类型的值。

## 示例

```Plain Text
mysql> select uuid();
+--------------------------------------+
| uuid()                               |
+--------------------------------------+
| 74a2ed19-9d21-4a99-a67b-aa5545f26454 |
+--------------------------------------+
1 row in set (0.01 sec)
```