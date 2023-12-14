---
displayed_sidebar: "英文"
---

# bitmap_and

## 说明

计算两个输入位图的交集并返回新的位图。

## 语法

```Haskell
BITMAP BITMAP_AND(BITMAP lhs, BITMAP rhs)
```

## 示例

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

## 关键字

BITMAP_AND,BITMAP