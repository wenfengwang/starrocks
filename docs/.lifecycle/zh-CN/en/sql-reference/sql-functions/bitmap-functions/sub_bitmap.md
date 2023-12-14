---
displayed_sidebar: "中文"
---

# sub_bitmap

## 描述

从`偏移量`指定的位置开始截取BITMAP值`src`的`len`个元素。输出的元素是`src`的子集。

此功能主要用于分页查询等场景。支持v2.5及以上版本。

此功能类似于[bitmap_subset_limit](./bitmap_subset_limit.md)。不同之处在于此功能从偏移量开始截取元素，而bitmap_subset_limit是从元素值(`start_range`)开始截取元素。

## 语法

```Haskell
BITMAP sub_bitmap(BITMAP src, BIGINT offset, BIGINT len)
```

## 参数

- `src`: 您希望获取元素的BITMAP值。
- `offset`: 起始位置。必须是BIGINT值。在使用`offset`时，请注意以下几点：
  - 偏移量从0开始。
  - 负偏移量从右向左计数。请参见示例3和4。
  - 如果由`offset`指定的起始位置超出了BITMAP值的实际长度，则返回NULL。请参见示例6。
- `len`: 要获取的元素个数。必须是大于或等于1的BIGINT值。如果匹配元素的个数少于`len`的值，则返回所有匹配元素。请参见示例2、3和7。

## 返回值

返回一个BITMAP类型的值。如果任何输入参数无效，则返回NULL。

## 示例

在以下示例中，sub_bitmap()的输入是[bitmap_from_string](./bitmap_from_string.md)的输出。例如，`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')`返回`1, 3, 5, 7, 9`。sub_bitmap()将此BITMAP值作为输入。

示例1：从偏移量设置为0的BITMAP值中获取两个元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

示例2：从偏移量设置为0的BITMAP值中获取100个元素。100超出了BITMAP值的长度，返回所有匹配元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

示例3：从偏移量设置为-3的BITMAP值中获取100个元素。100超出了BITMAP值的长度，返回所有匹配元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+
```

示例4：从偏移量设置为-3的BITMAP值中获取两个元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+
```

示例5：返回NULL，因为`-10`是`len`的无效输入。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例6：由偏移量5指定的起始位置超出了BITMAP值`1,3,5,7,9`的长度。返回NULL。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, 1)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例7：将`len`设置为5，但只有两个元素符合条件。返回这两个元素。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -2, 5)) value;
+-------+
| value |
+-------+
| 7,9   |
+-------+
```