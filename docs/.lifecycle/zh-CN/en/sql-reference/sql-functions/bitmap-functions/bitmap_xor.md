---
displayed_sidebar: "Chinese"
---

# bitmap_xor

## 描述

计算由 `lhs` 和 `rhs` 中独特元素组成的集合。这在逻辑上等同于 `bitmap_andnot(bitmap_or(lhs, rhs), bitmap_and(lhs, rhs))`（补集）。

## 语法

```Haskell
bitmap_xor(BITMAP lhs, BITMAP rhs)
```

## 示例

```plain text
mysql> select bitmap_to_string(bitmap_xor(bitmap_from_string('1, 3'), bitmap_from_string('2'))) cnt;
+------+
|cnt   |
+------+
|1,2,3 |
+------+
```

## 关键字

BITMAP_XOR, BITMAP