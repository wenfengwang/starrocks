---
displayed_sidebar: English
---

# UUID

## 描述

返回 VARCHAR 类型的随机 UUID。对该函数的两次调用可以生成两个不同的数字。UUID 的长度为 36 个字符，由 5 个十六进制数字组成，用四个连字符连接在一起，格式为 aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee。

## 语法

```Haskell
uuid();
```

## 参数

无

## 返回值

返回 VARCHAR 类型的值。

## 例子

```Plain Text
mysql> select uuid();
+--------------------------------------+
| uuid()                               |
+--------------------------------------+
| 74a2ed19-9d21-4a99-a67b-aa5545f26454 |
+--------------------------------------+
1 行受影响 (0.01 秒)