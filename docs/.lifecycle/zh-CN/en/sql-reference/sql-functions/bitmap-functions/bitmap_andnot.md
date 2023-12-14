---
displayed_sidebar: "Chinese"
---

# bitmap_andnot

## 描述

返回`lhs`中存在但`rhs`中不存在的位图值，并返回新的位图。

## 语法

```Haskell
bitmap_andnot(BITMAP lhs, BITMAP rhs)
```

## 示例

```plain text
mysql> select bitmap_to_string(bitmap_andnot(bitmap_from_string('1, 3'), bitmap_from_string('2'))) cnt;
+------+
|cnt   |
+------+
|1,3   |
+------+

mysql> select bitmap_to_string(bitmap_andnot(bitmap_from_string('1,3,5'), bitmap_from_string('1'))) cnt;
+------+
|cnt   |
+------+
|3,5   |
+------+
```

## 关键词

BITMAP_ANDNOT, BITMAP