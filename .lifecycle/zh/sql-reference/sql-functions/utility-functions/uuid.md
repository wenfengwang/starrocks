---
displayed_sidebar: English
---

# uuid

## 描述

返回一个随机的、VARCHAR 类型的 UUID。调用这个函数两次会生成两个不同的数值。UUID 的长度是 36 个字符，由 5 组十六进制数字组成，数字之间通过四个连字符相连接，格式为 aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee。

## 语法

```Haskell
uuid();
```

## 参数

无

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plain
mysql> select uuid();
+--------------------------------------+
| uuid()                               |
+--------------------------------------+
| 74a2ed19-9d21-4a99-a67b-aa5545f26454 |
+--------------------------------------+
1 row in set (0.01 sec)
```
