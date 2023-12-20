---
displayed_sidebar: English
---

# sub_bitmap

## 描述

从 `src` 的 BITMAP 值中，从 `offset` 指定的位置开始截取 `len` 个元素。输出的元素是 `src` 的子集。

此函数主要用于分页查询等场景。从 v2.5 版本开始支持。

此函数与 [bitmap_subset_limit](./bitmap_subset_limit.md) 相似。区别在于，sub_bitmap 从一个偏移量开始截取元素，而 bitmap_subset_limit 则是从一个元素值 (`start_range`) 开始截取元素。

## 语法

```Haskell
BITMAP sub_bitmap(BITMAP src, BIGINT offset, BIGINT len)
```

## 参数

- `src`：要获取元素的 BITMAP 值。
- `offset`：起始位置。必须是 BIGINT 类型的值。使用 `offset` 时请注意以下几点：
  - 偏移量从 0 开始。
  - 负偏移量从右向左计数。参见示例 3 和 4。
  - 如果 `offset` 指定的起始位置超过了 BITMAP 值的实际长度，则返回 NULL。参见示例 6。
- `len`：要获取的元素数量。必须是大于或等于 1 的 BIGINT 类型的值。如果匹配的元素数量少于 `len` 的值，则返回所有匹配的元素。参见示例 2、3 和 7。

## 返回值

返回 BITMAP 类型的值。如果任何输入参数无效，则返回 NULL。

## 示例

在以下示例中，sub_bitmap() 的输入是 [bitmap_from_string](./bitmap_from_string.md) 的输出。例如，`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` 返回 `1, 3, 5, 7, 9`。sub_bitmap() 使用这个 BITMAP 值作为输入。

示例 1：从偏移量设置为 0 的 BITMAP 值中获取两个元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

示例 2：从偏移量设置为 0 的 BITMAP 值中获取 100 个元素。由于 100 超出了 BITMAP 值的长度，返回所有匹配的元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

示例 3：从偏移量设置为 -3 的 BITMAP 值中获取 100 个元素。由于 100 超出了 BITMAP 值的长度，返回所有匹配的元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+
```

示例 4：从偏移量设置为 -3 的 BITMAP 值中获取两个元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+
```

示例 5：由于 `-10` 是 `len` 的无效输入，返回 NULL。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例 6：由于偏移量 5 指定的起始位置超过了 BITMAP 值 `1,3,5,7,9` 的长度，返回 NULL。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, 1)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例 7：`len` 设置为 5，但只有两个元素符合条件。返回这两个元素。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -2, 5)) value;
+-------+
| value |
+-------+
| 7,9   |
+-------+
```