---
displayed_sidebar: Chinese
---

# subdivide_bitmap

## 機能

大きな bitmap を複数の子 bitmap に分割します。

この関数は、bitmap をエクスポートする際に使用され、bitmap が大きすぎると MySQL プロトコルのパケットサイズの上限を超えることがあります。

この関数はバージョン 2.5 からサポートされています。

## 文法

```Haskell
BITMAP subdivide_bitmap(bitmap, length)
```

## パラメータ説明

`bitmap`: 分割する必要がある bitmap。必須。
`length`: 分割後のサイズ。各 bitmap の長さはこの値以下でなければなりません。必須。

## 戻り値の説明

`length` 以下のサイズに分割された複数の子 bitmap。

## 例

`t1` というテーブルがあり、`c2` 列が BITMAP 型であるとします。

```Plain
-- bitmap_to_string() 関数を使用して、複数行の Bitmap を1つの文字列に変換します。
mysql> select c1, bitmap_to_string(c2) from t1;
+------+----------------------+
| c1   | bitmap_to_string(c2) |
+------+----------------------+
|    1 | 1,2,3,4,5,6,7,8,9,10 |
+------+----------------------+

-- subdivide_bitmap() 関数を使用して、この Bitmap を長さが 3 以下の複数の Bitmap に分割します。その後、bitmap_to_string() を使用して、分割された複数行の Bitmap をクライアントに表示します。
mysql> select c1, bitmap_to_string(subdivide_bitmap) from t1, subdivide_bitmap(c2, 3);
+------+------------------------------------+
| c1   | bitmap_to_string(subdivide_bitmap) |
+------+------------------------------------+
|    1 | 1,2,3                              |
|    1 | 4,5,6                              |
|    1 | 7,8,9                              |
|    1 | 10                                 |
+------+------------------------------------+
```
