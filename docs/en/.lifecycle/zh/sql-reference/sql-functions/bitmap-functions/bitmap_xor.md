---
displayed_sidebar: English
---

# bitmap_xor

## 描述

计算由`lhs`和`rhs`中独有的元素组成的集合。它在逻辑上等价于`bitmap_andnot(bitmap_or(lhs, rhs), bitmap_and(lhs, rhs))`（补集）。

## 语法

```Haskell
bitmap_xor(BITMAP lhs, BITMAP rhs)
```

## 例子

```plain text
mysql> select bitmap_to_string(bitmap_xor(bitmap_from_string('1, 3'), bitmap_from_string('2'))) cnt;
+------+
|cnt   |
+------+
|1,2,3 |
+------+
```

## 关键词

BITMAP_XOR、BITMAP