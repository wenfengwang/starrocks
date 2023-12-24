---
displayed_sidebar: English
---

# bitmap_subset_in_range

## 描述

截取 Bitmap 值中`start_range`和`end_range`（不包括`start_range`）范围内的元素。输出的元素是 Bitmap 值的子集。

该函数主要用于分页查询等场景。从 v3.1 开始支持。

## 语法

```Haskell
BITMAP bitmap_subset_in_range(BITMAP src, BIGINT start_range, BIGINT end_range)
```

## 参数

- `src`：要获取元素的 Bitmap 值。
- `start_range`：截取元素的起始范围。它必须是 BIGINT 值。如果指定的起始范围超过 BITMAP 值的最大长度，则返回 NULL。参见示例 4。
- `end_range`：截取元素的结束范围。它必须是 BIGINT 值。如果`end_range`等于或小于`start_range`，则返回 NULL。参见示例 3。

## 返回值

返回 BITMAP 类型的值。如果任何输入参数无效，则返回 NULL。

## 使用说明

子集元素包括`start_range`但不包括`end_range`。参见示例 5。

## 例子

在以下示例中，bitmap_subset_in_range()的输入是[bitmap_from_string](./bitmap_from_string.md)的输出。例如，`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')`返回`1, 3, 5, 7, 9`。bitmap_subset_in_range()将此 BITMAP 值作为输入。

示例 1：从 BITMAP 值中获取元素值在范围1到4内的子集元素。此范围内的值为1和3。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

示例 2：从 BITMAP 值中获取元素值在范围1到100内的子集元素。结束值超过 BITMAP 值的最大长度，并返回所有匹配的元素。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

示例 3：返回 NULL，因为结束范围`3`小于起始范围`4`。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 4, 3)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例 4：起始范围10超出了 BITMAP 值的最大长度（5）。`1,3,5,7,9`返回 NULL。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例 5：返回的子集包含起始值`1`，但不包括结束值`3`。

```plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,4,5,6,7,9'), 1, 3)) value;
+-------+
| value |
+-------+
| 1     |
+-------+
```

## 引用

[bitmap_subset_limit](./bitmap_subset_limit.md)
