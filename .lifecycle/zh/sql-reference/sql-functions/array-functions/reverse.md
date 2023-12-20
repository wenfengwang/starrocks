---
displayed_sidebar: English
---

# 反转

## 描述

反转字符串或数组。返回一个字符串或数组，其中的字符或数组元素按相反的顺序排列。

## 语法

```Haskell
reverse(param)
```

## 参数

param：需要反转的字符串或数组。其类型可以是 VARCHAR、CHAR 或 ARRAY。

目前，此函数只支持一维数组，并且数组元素不能是 DECIMAL 类型。该函数支持以下类型的数组元素：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE 和 JSON。**从版本 2.5 开始支持 JSON 类型。**

## 返回值

返回值的类型与 param 参数相同。

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
