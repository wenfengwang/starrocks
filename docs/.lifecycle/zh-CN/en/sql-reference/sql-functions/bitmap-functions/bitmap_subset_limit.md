---
displayed_sidebar: "中文"
---

# bitmap_subset_limit

## 描述

从具有从`start_range`开始的元素值的BITMAP值中截取指定数量的元素。输出的元素是`src`的子集。

此函数主要用于分页查询等场景。支持v3.1及以上版本。

此函数类似于[sub_bitmap](./sub_bitmap.md)。不同之处在于，此函数截取从一个元素值(`start_range`)开始的元素，而sub_bitmap是从偏移量开始截取元素。

## 语法

```Haskell
BITMAP bitmap_subset_limit(BITMAP src, BIGINT start_range, BIGINT limit)
```

## 参数

- `src`：要获取元素的BITMAP值。
- `start_range`：截取元素的起始范围。必须是BIGINT值。如果指定的起始范围超过BITMAP值的最大元素且`limit`为正，则返回NULL。请参见示例4。
- `limit`：从`start_range`开始获取的元素数量。负限制从右到左计数。如果匹配元素的数量少于`limit`的值，则返回所有匹配元素。

## 返回值

返回BITMAP类型的值。如果任一输入参数无效，则返回NULL。

## 使用注意事项

- 子集元素包括`start range`。
- 负限制从右到左计数。请参见示例3。

## 示例

在以下示例中，bitmap_subset_limit()的输入是[bitmap_from_string](./bitmap_from_string.md)的输出。例如，`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')`返回`1, 3, 5, 7, 9`。bitmap_subset_limit()以此BITMAP值作为输入。

示例1：从元素值从1开始的BITMAP值中获取4个元素。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+---------+
|  value  |
+---------+
| 1,3,5,7 |
+---------+
```

示例2：从元素值从1开始的BITMAP值中获取100个元素。限制超过了BITMAP值的长度，返回所有匹配的元素。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

示例3：从元素值从5开始的BITMAP值中获取-2个元素（从右到左计数）。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, -2)) value;
+-----------+
| value     |
+-----------+
| 3,5       |
+-----------+
```

示例4：起始范围10超过了“1,3,5,7,9”这个BITMAP值的最大元素，并且limit为正。返回NULL。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```