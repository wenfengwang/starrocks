---
displayed_sidebar: Chinese
---

# bitmap_contains

## 機能

入力値がBitmap列に含まれているかどうかを計算します。

## 構文

```Haskell
BITMAP_CONTAINS(bitmap, input)
```

## パラメータ説明

`bitmap`: サポートされるデータ型は BITMAP です。

`input`: 入力値、サポートされるデータ型は BIGINT です。

## 戻り値の説明

戻り値のデータ型は BOOLEAN です。

## 例

```Plain Text
MySQL > select bitmap_contains(to_bitmap(1),2) as cnt;
+------+
| cnt  |
+------+
|    0 |
+------+

MySQL > select bitmap_contains(to_bitmap(1),1) as cnt;
+------+
| cnt  |
+------+
|    1 |
+------+
```
