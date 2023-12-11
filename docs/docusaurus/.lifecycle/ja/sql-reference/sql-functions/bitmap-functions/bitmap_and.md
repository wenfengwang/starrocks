---
displayed_sidebar: "Japanese"
---

# bitmap_and

## Description

2つの入力ビットマップの積集合を計算して新しいビットマップを返します。

## Syntax

```Haskell
BITMAP BITMAP_AND(BITMAP lhs, BITMAP rhs)
```

## Examples

```plain text
MySQL > select bitmap_count(bitmap_and(to_bitmap(1), to_bitmap(2))) cnt;
+------+
| cnt  |
+------+
|    0 |
+------+

MySQL > select bitmap_count(bitmap_and(to_bitmap(1), to_bitmap(1))) cnt;
+------+
| cnt  |
+------+
|    1 |
+------+
```

## keyword

BITMAP_AND,BITMAP