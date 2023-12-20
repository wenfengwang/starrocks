---
displayed_sidebar: English
---

# 位图包含

## 描述

判断输入值是否存在于位图列中，并返回布尔值。

## 语法

```Haskell
B00LEAN BITMAP_CONTAINS(BITMAP bitmap, BIGINT input)
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

## 关键词

BITMAP_CONTAINS，BITMAP
