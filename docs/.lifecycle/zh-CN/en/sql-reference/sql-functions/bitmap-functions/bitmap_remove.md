---
displayed_sidebar: "中文"
---

# bitmap_remove

## 描述

从位图值 `lhs` 中移除 `input`，并返回结果集。

## 语法

```Haskell
bitmap_remove(BITMAP lhs, BIGINT input)
```

## 示例

```plain text
mysql> select bitmap_to_string(**bitmap_remove**(bitmap_from_string('1, 3'), 3)) cnt;
+------+
|cnt   |
+------+
|1     |
+------+

mysql> select bitmap_to_string(**bitmap_remove**(bitmap_from_string('1,3,5'), 6)) cnt;
+------+
|cnt   |
+------+
|1,3,5 |
+------+
```

## 关键词

BITMAP_REMOVE, BITMAP