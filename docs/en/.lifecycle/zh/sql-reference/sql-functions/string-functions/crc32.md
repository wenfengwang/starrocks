---
displayed_sidebar: English
---

# crc32

## 描述

此函数返回字符串的CRC32校验和

## 语法

```Haskell
BIGINT crc32(VARCHAR str)
```

## 例子

```Plain Text
MySQL [(none)]> select crc32("starrocks");
+--------------------+
| crc32('starrocks') |
+--------------------+
|         2312449062 |
+--------------------+
```
```Plain Text
MySQL [(none)]> select crc32(null);
+-------------+
| crc32(NULL) |
+-------------+
|        NULL |
+-------------+
1 行受影响 (0.18 秒)
```

## 关键词

CRC32