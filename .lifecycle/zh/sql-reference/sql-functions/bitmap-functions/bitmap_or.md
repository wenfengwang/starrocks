---
displayed_sidebar: English
---

# 位图或运算

## 描述

计算两个输入位图的并集，并返回一个新的位图。

## 语法

```Haskell
BITMAP BITMAP_OR(BITMAP lhs, BITMAP rhs)
```

## 示例

```Plain
MySQL > select bitmap_count(bitmap_or(to_bitmap(1), to_bitmap(2))) cnt;
+------+
| cnt  |
+------+
|    2 |
+------+

MySQL > select bitmap_count(bitmap_or(to_bitmap(1), to_bitmap(1))) cnt;
+------+
| cnt  |
+------+
|    1 |
+------+
```

## 关键词

BITMAP_OR，位图
