---
displayed_sidebar: English
---

# json_array

## 描述

将SQL数组中的每个元素转换成JSON值，并返回一个包含这些JSON值的JSON数组。

## 语法

```Haskell
json_array(value, ...)
```

## 参数

value：SQL数组中的一个元素。只支持NULL值以及以下数据类型：STRING、VARCHAR、CHAR、JSON、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE、FLOAT和BOOLEAN。

## 返回值

返回一个JSON数组。

## 示例

示例1：构建一个包含不同数据类型值的JSON数组。

```plaintext
mysql> SELECT json_array(1, true, 'starrocks', 1.1);

       -> [1, true, "starrocks", 1.1]
```

示例2：构建一个空的JSON数组。

```plaintext
mysql> SELECT json_array();

       -> []
```
