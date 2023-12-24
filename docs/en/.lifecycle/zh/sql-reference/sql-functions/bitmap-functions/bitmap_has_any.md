---
displayed_sidebar: English
---

# bitmap_has_any

## 描述

计算两个 Bitmap 列之间是否存在相交的元素，并返回布尔值。

## 语法

```Haskell
B00LEAN BITMAP_HAS_ANY(BITMAP lhs, BITMAP rhs)
```

## 例子

```Plain Text
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

## 关键词

BITMAP_HAS_ANY，BITMAP