---
displayed_sidebar: English
---

# CRC32

## 描述

此函数用于返回一个字符串的CRC32校验和。

## 语法

```Haskell
BIGINT crc32(VARCHAR str)
```

## 示例

```Plain
MySQL [(none)]> select crc32("starrocks");
+--------------------+
| crc32('starrocks') |
+--------------------+
|         2312449062 |
+--------------------+
```
```Plain
MySQL [(none)]> select crc32(null);
+-------------+
| crc32(NULL) |
+-------------+
|        NULL |
+-------------+
1 row in set (0.18 sec)
```

## 关键字

CRC32
