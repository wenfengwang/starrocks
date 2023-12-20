---
displayed_sidebar: English
---

# 位图是否有交集

## 描述

计算两个位图（Bitmap）列之间是否有交集的元素，返回值是布尔型。

## 语法

```Haskell
B00LEAN BITMAP_HAS_ANY(BITMAP lhs, BITMAP rhs)
```

## 示例

```Plain
MySQL > select bitmap_has_any(to_bitmap(1),to_bitmap(2)) cnt;
+------+
| cnt  |
+------+
|    0 |
+------+

MySQL > select bitmap_has_any(to_bitmap(1),to_bitmap(1)) cnt;
+------+
| cnt  |
+------+
|    1 |
+------+
```

## 关键字

BITMAP_HAS_ANY，BITMAP
