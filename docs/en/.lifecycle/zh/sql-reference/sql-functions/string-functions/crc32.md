---
displayed_sidebar: English
---

# crc32

## 描述

此函数返回一个字符串的 CRC32 校验和

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
1行返回集 (0.18秒)
```

## 关键字

CRC32