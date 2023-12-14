---
displayed_sidebar: "Chinese"
---

# reverse

## 描述

反转字符串或数组。返回一个以相反顺序排列字符的字符串或数组。

## 语法

```Haskell
reverse(param)
```

## 参数

`param`: 要反转的字符串或数组。它可以是 VARCHAR、CHAR 或 ARRAY 类型。

目前，此函数仅支持一维数组，且数组元素不能是 DECIMAL 类型。此函数支持以下类型的数组元素：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE 和 JSON。**从2.5版本开始支持JSON。**

## 返回值

返回类型与 `param` 相同。

## 示例

示例 1：反转字符串。

```Plain Text
MySQL > SELECT REVERSE('hello');
+------------------+
| REVERSE('hello') |
+------------------+
| olleh            |
+------------------+
1 row in set (0.00 sec)
```

示例 2：反转数组。

```Plain Text
MYSQL> SELECT REVERSE([4,1,5,8]);
+--------------------+
| REVERSE([4,1,5,8]) |
+--------------------+
| [8,5,1,4]          |
+--------------------+
```