---
displayed_sidebar: English
---

# 数组交集

## 描述

返回一个数组，包含了一个或多个数组的交集元素。

## 语法

```Haskell
array_intersect(input0, input1, ...)
```

## 参数

输入：想要获取交集的一个或多个数组。请按照 (input0, input1, ...) 的格式指定数组，并确保这些数组的数据类型相同。

## 返回值

返回一个数组，其数据类型与指定的数组相同。

## 示例

示例 1：

```Plain
mysql> SELECT array_intersect(["SQL", "storage"], ["mysql", "query", "SQL"], ["SQL"])
AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| ["SQL"]      |
+--------------+
```

示例 2：

```Plain
mysql> SELECT array_intersect(["SQL", "storage"], ["mysql", null], [null]) AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| []           |
+--------------+
```

示例 3：

```Plain
mysql> SELECT array_intersect(["SQL", null, "storage"], ["mysql", null], [null]) AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| [null]       |
+--------------+
```
