---
displayed_sidebar: "中文"
---

# bitmap_has_any

## 描述

计算两个位图列之间是否存在交叉元素，并返回布尔值。

## 语法

```Haskell
B00LEAN BITMAP_HAS_ANY(BITMAP lhs, BITMAP rhs)
```

## 示例

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

## 关键字

BITMAP_HAS_ANY,BITMAP