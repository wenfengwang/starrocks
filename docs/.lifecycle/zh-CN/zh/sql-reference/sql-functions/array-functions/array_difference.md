---
displayed_sidebar: "Chinese"
---

# array_difference

## 功能

For numerical arrays, return an array consisting of the differences between adjacent elements (subtracting the former from the latter).

## 语法

```Haskell
output array_difference(input)
```

## 参数说明

* input: Numerical array.

## 返回值说明

Array type (consistent with the input input), containing the differences between adjacent elements in the input input, with the same length as the input.

## 示例

**示例一**:

```plain text
mysql> SELECT array_difference([342, 32423, 213, 23432]);
+-----------------------------------------+
| array_difference([342,32423,213,23432]) |
+-----------------------------------------+
| [0,32081,-32210,23219]                  |
+-----------------------------------------+
```

**示例二**:

```plain text
mysql> SELECT array_difference([342, 32423, 213, null, 23432]);
+----------------------------------------------+
| array_difference([342,32423,213,NULL,23432]) |
+----------------------------------------------+
| [0,32081,-32210,null,null]                   |
+----------------------------------------------+
```

**示例 三**:

```plain text
mysql> SELECT array_difference([1.2, 2.3, 3.2, 4324242.55]);
+--------------------------------------------+
| array_difference([1.2,2.3,3.2,4324242.55]) |
+--------------------------------------------+
| [0,1.1,0.9,4324239.35]                     |
+--------------------------------------------+
```

**示例 四**:

```plain text
mysql> SELECT array_difference([false, true, false]);
+----------------------------------------+
| array_difference([FALSE, TRUE, FALSE]) |
+----------------------------------------+
| [0,1,-1]                               |
+----------------------------------------+