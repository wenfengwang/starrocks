---
displayed_sidebar: English
---

# crc32

## 説明

この関数は、文字列のCRC32チェックサムを返します

## 構文

```Haskell
BIGINT crc32(VARCHAR str)
```

## 例

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
1行がセットされました (0.18秒)
```

## キーワード

CRC32
