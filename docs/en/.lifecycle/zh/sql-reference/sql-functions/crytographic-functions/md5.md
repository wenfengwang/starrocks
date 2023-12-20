---
displayed_sidebar: English
---

# MD5

使用 MD5 消息摘要算法来计算字符串的 128 位校验和。该校验和由 32 个字符的十六进制字符串表示。

## 语法

```sql
md5(expr)
```

## 参数

`expr`：要计算的字符串。必须是 VARCHAR 类型。

## 返回值

返回一个 VARCHAR 类型的校验和，为一个 32 字符的十六进制字符串。

如果输入为 NULL，则返回 NULL。

## 示例

```sql
select md5('abc');
```

```plaintext
+----------------------------------+
| md5('abc')                       |
+----------------------------------+
| 900150983cd24fb0d6963f7d28e17f72 |
+----------------------------------+
1 row in set (0.01 sec)
```

```sql
select md5(null);
```

```plaintext
+-----------+
| md5(NULL) |
+-----------+
| NULL      |
+-----------+
1 row in set (0.00 sec)
```

## 关键字

MD5, ENCRYPTION