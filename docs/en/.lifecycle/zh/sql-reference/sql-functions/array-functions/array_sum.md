---
displayed_sidebar: English
---

# array_sum

## 描述

对数组中的所有元素求和。

从 StarRocks 2.5 开始，array_sum() 可以接受 lambda 表达式作为参数。但是，它不能直接与 Lambda 表达式一起工作。它必须在 [array_map()](./array_map.md) 转换结果的基础上进行操作。

## 语法

```Haskell
array_sum(array(type))
array_sum(lambda_function, arr1, arr2...) = array_sum(array_map(lambda_function, arr1, arr2...))
```

## 参数

- `array(type)`: 您想要计算总和的数组。数组元素支持以下数据类型：BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, FLOAT, DOUBLE 和 DECIMALV2。
- `lambda_function`: 用于计算 array_sum() 目标数组的 lambda 表达式。

## 返回值

返回一个数值类型的结果。

## 示例

### 不使用 lambda 函数的 array_sum 使用方法

```plain
mysql> select array_sum([11, 11, 12]);
+-----------------------+
| array_sum([11, 11, 12]) |
+-----------------------+
| 34                    |
+-----------------------+

mysql> select array_sum([11.33, 11.11, 12.324]);
+---------------------------------+
| array_sum([11.33, 11.11, 12.324]) |
+---------------------------------+
| 34.764                          |
+---------------------------------+
```

### 结合 lambda 函数使用 array_sum

```plain
-- 将 [1,2,3] 与 [1,2,3] 相乘并求和。
select array_sum(array_map(x -> x * x, [1, 2, 3]));
+---------------------------------------------+
| array_sum(array_map(x -> x * x, [1, 2, 3])) |
+---------------------------------------------+
|                                          14 |
+---------------------------------------------+
```

## 关键字

ARRAY_SUM, ARRAY