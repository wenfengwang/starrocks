---
displayed_sidebar: English
---

# array_intersect

## 描述

返回一个或多个数组的交集中的元素所组成的数组。

## 语法

```Haskell
array_intersect(input0, input1, ...)
```

## 参数

`input`：一个或多个要获取其交集的数组。请以 `(input0, input1, ...)` 的格式指定数组，并确保你指定的数组具有相同的数据类型。

## 返回值

返回一个与你指定的数组具有相同数据类型的数组。

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