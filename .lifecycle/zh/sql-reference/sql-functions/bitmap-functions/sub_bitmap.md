---
displayed_sidebar: English
---

# 子位图

## 描述

从位图值src中，从由offset指定的位置开始截取len个元素。输出的元素是src的子集。

此函数主要用于分页查询等场景。自v2.5版本起提供支持。

此函数与[bitmap_subset_limit](./bitmap_subset_limit.md)相似。区别在于，此函数从一个偏移量开始截取元素，而bitmap_subset_limit则从一个元素值（`start_range`）开始截取元素。

## 语法

```Haskell
BITMAP sub_bitmap(BITMAP src, BIGINT offset, BIGINT len)
```

## 参数

- src：要获取元素的位图值。
- offset：起始位置。它必须是BIGINT类型的值。使用offset时，请注意以下几点：
  - 偏移量从0开始计数。
  - 负偏移量从右向左计数。参见示例3和4。
  - 如果offset指定的起始位置超出了位图值的实际长度，则返回NULL。参见示例6。
- len：要获取的元素个数。它必须是大于或等于1的BIGINT类型的值。如果匹配的元素个数少于len的值，则返回所有匹配的元素。参见示例2、3和7。

## 返回值

返回位图类型的值。如果任何输入参数无效，则返回NULL。

## 示例

在以下示例中，sub_bitmap()的输入是[bitmap_from_string](./bitmap_from_string.md)的输出。例如，`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')`返回`1, 3, 5, 7, 9`。sub_bitmap()以此位图值作为输入。

示例1：从偏移量设置为0的位图值中获取两个元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

示例2：从偏移量设置为0的位图值中获取100个元素。由于100超过了位图值的长度，因此返回所有匹配的元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

示例3：从偏移量设置为-3的位图值中获取100个元素。由于100超过了位图值的长度，因此返回所有匹配的元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+
```

示例4：从偏移量设置为-3的位图值中获取两个元素。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+
```

示例5：由于-10是len的无效输入，因此返回NULL。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例6：由于偏移量5指定的起始位置超过了位图值1, 3, 5, 7, 9的长度，因此返回NULL。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, 1)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

示例7：len设置为5，但只有两个元素符合条件。因此，返回这两个元素。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -2, 5)) value;
+-------+
| value |
+-------+
| 7,9   |
+-------+
```
