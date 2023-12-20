---
displayed_sidebar: English
---

# 数组求和

## 描述

计算数组中所有元素的总和。

从 StarRocks 2.5 版本开始，array_sum() 函数可以接受 lambda 表达式作为参数。但是，它不能直接与 Lambda 表达式共同作用。它必须在 [array_map()](./array_map.md) 转换结果的基础上进行操作。

## 语法

```Haskell
array_sum(array(type))
array_sum(lambda_function, arr1,arr2...) = array_sum(array_map(lambda_function, arr1,arr2...))
```

## 参数

- array(type)：你想要计算总和的数组。数组元素支持以下数据类型：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE 和 DECIMALV2。
- lambda_function：用于计算目标数组以供 array_sum() 函数使用的 lambda 表达式。

## 返回值

返回数值类型的结果。

## 示例

### 不使用 lambda 函数的情况下使用 array_sum

```plain
mysql> select array_sum([11, 11, 12]);
+-----------------------+
| array_sum([11,11,12]) |
+-----------------------+
| 34                    |
+-----------------------+

mysql> select array_sum([11.33, 11.11, 12.324]);
+---------------------------------+
| array_sum([11.33,11.11,12.324]) |
+---------------------------------+
| 34.764                          |
+---------------------------------+
```

### 结合 lambda 函数使用 array_sum

```plain
-- Multiply [1,2,3] by [1,2,3] and sum the elements.
select array_sum(array_map(x->x*x,[1,2,3]));
+---------------------------------------------+
| array_sum(array_map(x -> x * x, [1, 2, 3])) |
+---------------------------------------------+
|                                          14 |
+---------------------------------------------+
```

## 关键字

ARRAY_SUM, 数组
