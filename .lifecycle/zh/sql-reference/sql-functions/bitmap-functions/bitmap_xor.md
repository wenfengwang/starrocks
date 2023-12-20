---
displayed_sidebar: English
---

# 位图异或

## 描述

计算只存在于lhs和rhs中的唯一元素所构成的集合。逻辑上，这相当于对bitmap_or(lhs, rhs)和bitmap_and(lhs, rhs)的结果进行bitmap_andnot操作（即求补集）。

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

## 关键词

BITMAP_XOR，位图
