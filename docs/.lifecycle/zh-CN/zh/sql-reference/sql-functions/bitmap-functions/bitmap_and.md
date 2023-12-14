---
displayed_sidebar: "Chinese"
---

# bitmap_and

## 功能

计算两个位图的交集，返回新的位图。

## 语法

```Haskell
BITMAP_AND(lhs, rhs)
```

## 参数说明

`lhs`: 支持的数据类型为位图。

`rhs`: 支持的数据类型为位图。

## 返回值说明

返回值的数据类型为位图。

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