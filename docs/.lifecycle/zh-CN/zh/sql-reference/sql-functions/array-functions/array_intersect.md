---
displayed_sidebar: "中文"
---

# array_intersect

## 功能

针对多个同类型数组，返回它们的交集。

## 语法

```Haskell
output array_intersect(input0, input1, ...)
```

## 参数说明

* input：数量不限（至少1个）的类型相同的数组(input0, input1, ...)，数组中元素的具体类型可以是任意的。

## 返回值说明

返回值类型为Array（元素类型与输入中的数组元素类型一致），内容为所有输入数组(input0, input1, ...)的交集。

## 示例

**示例一**:

```plain text
mysql> SELECT array_intersect(["SQL", "storage"], ["mysql", "query", "SQL"], ["SQL"])
AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| ["SQL"]      |
+--------------+
```

**示例二**:

```plain text
mysql> SELECT array_intersect(["SQL", "storage"], ["mysql", null], [null]) AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| []           |
+--------------+
```

**示例三**:

```plain text
mysql> SELECT array_intersect(["SQL", null, "storage"], ["mysql", null], [null]) AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| [null]       |
+--------------+
```