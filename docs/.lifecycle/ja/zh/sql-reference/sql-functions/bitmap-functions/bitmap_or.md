---
displayed_sidebar: Chinese
---

# bitmap_or

## 機能

二つの入力された bitmap の和集合を計算し、新しい bitmap を返します。和集合とは、二つの集合の全ての要素を合わせて構成される集合のことを指します。重複する要素は一度だけカウントされます。

## 文法

```Haskell
BITMAP_OR(lhs, rhs)
```

## パラメータ説明

`lhs`: サポートされるデータ型は BITMAP です。

`rhs`: サポートされるデータ型は BITMAP です。

## 戻り値の説明

戻り値のデータ型は BITMAP です。

## 例

二つの bitmap の和集合を計算し、和集合に含まれる要素の総数を返します。

```Plain Text
select bitmap_count(bitmap_or(to_bitmap(1), to_bitmap(2))) cnt;
+------+
| cnt  |
+------+
|    2 |
+------+

select bitmap_count(bitmap_or(to_bitmap(1), to_bitmap(1))) cnt;
+------+
| cnt  |
+------+
|    1 |
+------+
```
