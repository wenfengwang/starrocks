---
displayed_sidebar: English
---

# bitmap_subset_in_range

## 描述

从 Bitmap 值中截取 `start_range` 和 `end_range`（不包括）范围内的元素。输出的元素是 Bitmap 值的子集。

此函数主要用于分页查询等场景。自 v3.1 版本起支持。

## 语法

```Haskell
BITMAP bitmap_subset_in_range(BITMAP src, BIGINT start_range, BIGINT end_range)
```

## 参数

- `src`：要从中获取元素的 Bitmap 值。
- `start_range`：截取元素的起始范围。必须是 BIGINT 类型的值。如果指定的起始范围超出了 BITMAP 值的最大长度，则返回 NULL。见示例 4。
- `end_range`：截取元素的结束范围。必须是 BIGINT 类型的值。如果 `end_range` 小于或等于 `start_range`，则返回 NULL。见示例 3。

## 返回值

返回 BITMAP 类型的值。如果任何输入参数无效，则返回 NULL。

## 使用说明

子集元素包括 `start_range` 但不包括 `end_range`。见示例 5。

## 示例

在以下示例中，`bitmap_subset_in_range()` 的输入是 [bitmap_from_string](./bitmap_from_string.md) 的输出。例如，`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` 返回 `1, 3, 5, 7, 9`。`bitmap_subset_in_range()` 使用这个 BITMAP 值作为输入。

示例 1：从 BITMAP 值中获取元素值在 1 到 4 范围内的子集元素，该范围内的值有 1 和 3。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

示例 2：从元素值在 1 到 100 范围内的 BITMAP 值中获取子集元素。结束值超过了 BITMAP 值的最大长度，因此返回所有匹配的元素。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

示例 3：因为结束范围 `3` 小于起始范围 `4`，所以返回 NULL。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 4, 3)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例 4：起始范围 10 超过了 BITMAP 值 `1,3,5,7,9` 的最大长度（5）。返回 NULL。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例 5：返回的子集包括起始值 `1` 但不包括结束值 `3`。

```plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,4,5,6,7,9'), 1, 3)) value;
+-------+
| value |
+-------+
| 1     |
+-------+
```

## 参考资料

[bitmap_subset_limit](./bitmap_subset_limit.md)