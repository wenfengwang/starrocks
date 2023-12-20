---
displayed_sidebar: English
---

# 位图移除

## 描述

从位图值 lhs 中移除输入并返回结果集。

## 语法

```Haskell
bitmap_remove(BITMAP lhs, BIGINT input)
```

## 示例

```plain
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

## 关键字

BITMAP_REMOVE，BITMAP
