---
displayed_sidebar: "Japanese"
---

# bitmap_has_any

## 説明

2つのビットマップ列の間に重なる要素があるかどうかを計算し、返り値はブール値です。

## 構文

```Haskell
B00LEAN BITMAP_HAS_ANY(BITMAP lhs, BITMAP rhs)
```

## 例

```Plain Text
MySQL > select bitmap_has_any(to_bitmap(1),to_bitmap(2)) cnt;
+------+
| cnt  |
+------+
|    0 |
+------+

MySQL > select bitmap_has_any(to_bitmap(1),to_bitmap(1)) cnt;
+------+
| cnt  |
+------+
|    1 |
+------+
```

## キーワード

BITMAP_HAS_ANY,BITMAP
