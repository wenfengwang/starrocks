---
displayed_sidebar: English
---

# bitmap_xor

## 描述

计算只存在于 `lhs` 和 `rhs` 中的唯一元素集合。逻辑上等同于 `bitmap_andnot(bitmap_or(lhs, rhs), bitmap_and(lhs, rhs))`（补集）。

## 语法

```Haskell
bitmap_xor(BITMAP lhs, BITMAP rhs)
```

## 示例

```plain
mysql> select bitmap_to_string(bitmap_xor(bitmap_from_string('1, 3'), bitmap_from_string('2'))) cnt;
+------+
|cnt   |
+------+
|1,2,3 |
+------+
```

## 关键字

BITMAP_XOR, BITMAP