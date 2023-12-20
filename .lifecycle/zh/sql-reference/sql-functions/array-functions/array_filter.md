---
displayed_sidebar: English
---

# 数组过滤

## 描述

返回与给定过滤条件匹配的数组中的值。

此函数有以下两种形式。Lambda形式允许进行更灵活的数组过滤。有关Lambda函数的更多信息，请参见[Lambda表达式](../Lambda_expression.md)。该函数自v2.5版本起得到支持。

## 语法

```Haskell
array_filter(array, array<bool>)
array_filter(lambda_function, arr1,arr2...)
```

- array_filter(array, array<bool>)

  返回与 array<bool> 匹配的数组中的值。

- array_filter(lambda_function, arr1, arr2...)

  返回与 Lambda 函数匹配的数组中的值。

## 参数

array：要过滤值的数组。

array<bool>：用来过滤值的表达式。

lambda_function：用来过滤值的 Lambda 函数。

## 使用说明

- array_filter(array, array<bool>) 的两个输入参数必须是数组，过滤表达式的结果应为 array<bool>。
- 在 `array_filter(lambda_function, arr1,arr2...)` 中的 Lambda 函数应遵循[array_map()](array_map.md)中的使用说明。
- 如果输入数组为 null，则返回 null。如果过滤数组为 null，则返回一个空数组。

## 示例

- 未使用 Lambda 函数的示例

  ```Plain
  -- All the elements in the array match the filter.
  select array_filter([1,2,3],[1,1,1]);
  +------------------------------------+
  | array_filter([1, 2, 3], [1, 1, 1]) |
  +------------------------------------+
  | [1,2,3]                            |
  +------------------------------------+
  1 row in set (0.01 sec)
  
  -- The filter is null and an empty array is returned.
  select array_filter([1,2,3],null);
  +-------------------------------+
  | array_filter([1, 2, 3], NULL) |
  +-------------------------------+
  | []                            |
  +-------------------------------+
  1 row in set (0.01 sec)
  
  -- The input array is null, null is returned.
  select array_filter(null,[1]);
  +-------------------------+
  | array_filter(NULL, [1]) |
  +-------------------------+
  | NULL                    |
  +-------------------------+
  
  -- Both the input array and filter are null. Null is returned.
  select array_filter(null,null);
  +--------------------------+
  | array_filter(NULL, NULL) |
  +--------------------------+
  | NULL                     |
  +--------------------------+
  1 row in set (0.01 sec)
  
  -- The filter contains a null element and an empty array is returned.
  select array_filter([1,2,3],[null]);
  +---------------------------------+
  | array_filter([1, 2, 3], [NULL]) |
  +---------------------------------+
  | []                              |
  +---------------------------------+
  1 row in set (0.01 sec)
  
  -- The filter contains two null elements and an empty array is returned.
  select array_filter([1,2,3],[null,null]);
  +---------------------------------------+
  | array_filter([1, 2, 3], [NULL, NULL]) |
  +---------------------------------------+
  | []                                    |
  +---------------------------------------+
  1 row in set (0.00 sec)
  
  -- Only one element matches the filter.
  select array_filter([1,2,3],[null,1,0]);
  +---------------------------------------+
  | array_filter([1, 2, 3], [NULL, 1, 0]) |
  +---------------------------------------+
  | [2]                                   |
  +---------------------------------------+
  1 row in set (0.00 sec)
  ```

- 使用 Lambda 函数的示例

  ```Plain
    -- Return the elements in x that are less than the elements in y.
    select array_filter((x,y) -> x < y, [1,2,null], [4,5,6]);
    +--------------------------------------------------------+
    | array_filter((x, y) -> x < y, [1, 2, NULL], [4, 5, 6]) |
    +--------------------------------------------------------+
    | [1,2]                                                  |
    +--------------------------------------------------------+
  ```
