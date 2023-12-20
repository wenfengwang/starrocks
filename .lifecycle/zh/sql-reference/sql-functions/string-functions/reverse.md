---
displayed_sidebar: English
---

# 反转

## 描述

反转字符串或数组。返回一个字符串或数组，其字符或元素顺序被颠倒。

## 语法

```Haskell
reverse(param)
```

## 参数

param：需要反转的字符串或数组。其类型可以是 VARCHAR、CHAR 或 ARRAY。

目前，该函数只支持一维数组，并且数组元素不能是 DECIMAL 类型。该函数支持以下类型的数组元素：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE 和 JSON。**JSON**类型从 2.5 版本开始支持。

## 返回值

返回值类型与 param 相同。

## 示例

示例 1：反转一个字符串。

```Plain
MySQL > SELECT REVERSE('hello');
+------------------+
| REVERSE('hello') |
+------------------+
| olleh            |
+------------------+
1 row in set (0.00 sec)
```

示例 2：反转一个数组。

```Plain
MYSQL> SELECT REVERSE([4,1,5,8]);
+--------------------+
| REVERSE([4,1,5,8]) |
+--------------------+
| [8,5,1,4]          |
+--------------------+
```
