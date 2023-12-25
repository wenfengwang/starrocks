---
displayed_sidebar: Chinese
---

# bitmap_has_any

## 機能

二つの Bitmap 列に共通する要素が存在するかどうかを計算し、結果は Boolean 値で返されます。

## 文法

```Haskell
BITMAP_HAS_ANY(lhs, rhs)
```

## パラメータ説明

`lhs`: 対応するデータ型は BITMAP です。

`rhs`: 対応するデータ型は BITMAP です。

## 戻り値の説明

戻り値のデータ型は BOOLEAN です。

## 例

```Plain Text
MySQL > select bitmap_has_any(to_bitmap(1),to_bitmap(2)) as cnt;
+------+
| cnt  |
+------+
|    0 |
+------+

MySQL > select bitmap_has_any(to_bitmap(1),to_bitmap(1)) as cnt;
+------+
| cnt  |
+------+
|    1 |
+------+
```
