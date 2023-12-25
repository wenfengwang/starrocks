---
displayed_sidebar: Chinese
---

# bitmap_remove

## 機能

Bitmap から指定された数値を削除します。

## 構文

```Haskell
bitmap_remove(lhs, input)
```

## パラメータ説明

`lhs`: サポートされるデータ型は BITMAP です。

`input`: サポートされるデータ型は BIGINT です。

## 戻り値の説明

戻り値のデータ型は BITMAP です。

## 例

```plain text
mysql> select bitmap_to_string(bitmap_remove(bitmap_from_string('1, 3'), 3)) cnt;
+------+
|cnt   |
+------+
|1     |
+------+

mysql> select bitmap_to_string(bitmap_remove(bitmap_from_string('1,3,5'), 6)) cnt;
+------+
|cnt   |
+------+
|1,3,5 |
+------+
```
