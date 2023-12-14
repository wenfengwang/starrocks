```yaml
---
displayed_sidebar: "Chinese"
---

# bitmap_contains

## 描述

计算输入值是否在位图列中，并返回布尔值。

## 语法

```Haskell
B00LEAN BITMAP_CONTAINS(BITMAP 位图, BIGINT 输入值)
```

## 示例

```Plain Text
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

BITMAP_CONTAINS, 位图