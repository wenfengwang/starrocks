---
displayed_sidebar: English
---

# map_from_arrays

## 描述

从给定的键数组和值数组中创建 MAP 值。

该函数从 v3.1 版本开始支持。

## 语法

```Haskell
MAP map_from_arrays(ARRAY keys, ARRAY values)
```

## 参数

- `keys`：用于构建结果 MAP 的键。确保 keys 的元素是唯一的。
- `values`：用于构建结果 MAP 的值。

## 返回值

返回由输入的 keys 和 values 构建的 MAP。

- keys 和 values 必须长度相同，否则会返回错误。
- 如果 key 或 value 为 NULL，则此函数返回 NULL。
- 返回的 MAP 值拥有唯一的键。

## 示例

```Plaintext
select map_from_arrays([1, 2], ['Star', 'Rocks']);
+--------------------------------------------+
| map_from_arrays([1, 2], ['Star', 'Rocks']) |
+--------------------------------------------+
| {1:"Star",2:"Rocks"}                       |
+--------------------------------------------+
```

```Plaintext
select map_from_arrays([1, 2], NULL);
+-------------------------------+
| map_from_arrays([1, 2], NULL) |
+-------------------------------+
| NULL                          |
+-------------------------------+

select map_from_arrays([1, 3, NULL, 2, NULL], ['ab', 'cdd', NULL, NULL, 'abc']);
+--------------------------------------------------------------------------+
| map_from_arrays([1, 3, NULL, 2, NULL], ['ab', 'cdd', NULL, NULL, 'abc']) |
+--------------------------------------------------------------------------+
| {1:"ab", 3:"cdd", 2:NULL, NULL:"abc"}                                   |
+--------------------------------------------------------------------------+
```