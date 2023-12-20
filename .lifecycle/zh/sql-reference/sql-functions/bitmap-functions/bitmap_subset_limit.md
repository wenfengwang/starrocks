---
displayed_sidebar: English
---

# 位图子集限制

## 描述

从 BITMAP 值中截取指定数量的元素，这些元素的值从起始范围开始。输出的元素是源 BITMAP 的一个子集。

该函数主要应用于分页查询等场景。自 v3.1 版本起提供支持。

此函数与 [sub_bitmap](./sub_bitmap.md) 相似。区别在于，此函数从一个元素值（`start_range`）开始截取元素，而 sub_bitmap 则是从一个偏移量开始截取元素。

## 语法

```Haskell
BITMAP bitmap_subset_limit(BITMAP src, BIGINT start_range, BIGINT limit)
```

## 参数

- src：要从中提取元素的 BITMAP 值。
- start_range：开始截取元素的起始范围。它必须是 BIGINT 类型的值。如果指定的起始范围超出了 BITMAP 值的最大元素，并且 limit 是正值，则返回 NULL。请参见示例 4。
- limit：从 start_range 开始要获取的元素数量。负数 limit 表示从右向左计数。如果匹配元素的数量少于 limit 的值，则返回所有匹配的元素。

## 返回值

返回 BITMAP 类型的值。如果任何输入参数无效，则返回 NULL。

## 使用说明

- 子集元素包括起始范围。
- 负数 limit 表示从右向左计数。请参见示例 3。

## 示例

在以下示例中，bitmap_subset_limit() 的输入是 [bitmap_from_string](./bitmap_from_string.md) 的输出。例如，`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` 返回 `1, 3, 5, 7, 9`。bitmap_subset_limit() 使用这个 BITMAP 值作为输入。

示例 1：从元素值从 1 开始的 BITMAP 值中获取 4 个元素。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+---------+
|  value  |
+---------+
| 1,3,5,7 |
+---------+
```

示例 2：从元素值从 1 开始的 BITMAP 值中获取 100 个元素。由于 limit 超出了 BITMAP 值的长度，因此返回所有匹配的元素。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

示例 3：从元素值从 5 开始的 BITMAP 值中获取 -2 个元素（从右向左计数）。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, -2)) value;
+-----------+
| value     |
+-----------+
| 3,5       |
+-----------+
```

示例 4：起始范围 10 超出了 BITMAP 值 1,3,5,7,9 的最大元素，并且 limit 是正值。因此返回 NULL。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```
