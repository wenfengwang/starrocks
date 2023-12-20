---
displayed_sidebar: English
---

# bitmap_has_any

## 描述

计算两个Bitmap列之间是否有交集元素，返回值是布尔值。

## 语法

```Haskell
BOOLEAN BITMAP_HAS_ANY(BITMAP lhs, BITMAP rhs)
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

BITMAP_HAS_ANY, BITMAP