---
displayed_sidebar: English
---

# bitmap_contains

## 描述

计算输入值是否在bitmap列中，并返回一个布尔值。

## 语法

```Haskell
BOOLEAN BITMAP_CONTAINS(BITMAP bitmap, BIGINT input)
```

## 示例

```Plain
MySQL > select bitmap_contains(to_bitmap(1),2) cnt;
+------+
| cnt  |
+------+
|    0 |
+------+

MySQL > select bitmap_contains(to_bitmap(1),1) cnt;
+------+
| cnt  |
+------+
|    1 |
+------+
```

## 关键字

BITMAP_CONTAINS,BITMAP