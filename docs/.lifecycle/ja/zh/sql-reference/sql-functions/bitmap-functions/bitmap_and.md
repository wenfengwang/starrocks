---
displayed_sidebar: Chinese
---

# bitmap_and

## 機能

二つのビットマップの交差部分を計算し、新しいビットマップを返します。

## 文法

```Haskell
BITMAP_AND(lhs, rhs)
```

## パラメータ説明

`lhs`: サポートされるデータ型は BITMAP です。

`rhs`: サポートされるデータ型は BITMAP です。

## 戻り値の説明

戻り値のデータ型は BITMAP です。

## 例

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
