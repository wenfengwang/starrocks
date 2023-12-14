---
displayed_sidebar: "Chinese"
---

# array_sum

## 描述

对数组中所有元素求和。

从 StarRocks 2.5 开始，array_sum() 可以接受一个 lambda 表达式作为参数。但是，它不能直接与 Lambda 表达式一起使用，必须在从 [array_map()](./array_map.md) 转换的结果上使用。

## 语法

```Haskell
array_sum(array(type))
array_sum(lambda_function, arr1,arr2...) = array_sum(array_map(lambda_function, arr1,arr2...))
```

## 参数

- `array(type)`: 要计算其总和的数组。数组元素支持以下数据类型: BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, FLOAT, DOUBLE, 和 DECIMALV2。
- `lambda_function`: 用于计算 array_sum() 目标数组的 lambda 表达式。

## 返回值

返回数值类型的值。

## 示例

### 使用不带 lambda 函数的 array_sum

```plain text
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

### 使用带 lambda 函数的 array_sum

```plain text
-- 将 [1,2,3] 乘以 [1,2,3] 并对元素求和。
select array_sum(array_map(x->x*x,[1,2,3]));
+---------------------------------------------+
| array_sum(array_map(x -> x * x, [1, 2, 3])) |
+---------------------------------------------+
|                                          14 |
+---------------------------------------------+
```

## 关键词

ARRAY_SUM,ARRAY