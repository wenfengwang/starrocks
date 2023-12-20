---
displayed_sidebar: English
---

# uuid

## 描述

返回一个随机的 UUID，类型为 VARCHAR。两次调用此函数会生成两个不同的数值。UUID 的长度是 36 个字符，包含 5 组十六进制数字，这些数字由四个连字符连接，格式为 aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee。

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