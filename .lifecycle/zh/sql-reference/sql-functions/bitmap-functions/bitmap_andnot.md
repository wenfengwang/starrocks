---
displayed_sidebar: English
---

# 位图且非

## 描述

返回仅存在于左侧位图（lhs）而不在右侧位图（rhs）中的值，并生成一个新的位图。

## 语法

```Haskell
bitmap_andnot(BITMAP lhs, BITMAP rhs)
```

## 示例

```plain
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

## 关键字

BITMAP_ANDNOT，位图
