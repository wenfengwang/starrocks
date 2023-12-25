---
displayed_sidebar: English
---

# bitmap_subset_in_range

## 説明

`start_range` から `end_range`（この範囲を含まない）までの範囲内の Bitmap 値から要素を抽出します。出力要素は Bitmap 値のサブセットです。

この関数は、ページネーションされたクエリなどのシナリオで主に使用されます。v3.1 からサポートされています。

## 構文

```Haskell
BITMAP bitmap_subset_in_range(BITMAP src, BIGINT start_range, BIGINT end_range)
```

## パラメータ

- `src`: 要素を取得する元の Bitmap 値。
- `start_range`: 要素を抽出する開始範囲。BIGINT 値でなければなりません。指定された開始範囲が BITMAP 値の最大長を超えた場合、NULL が返されます。例 4 を参照してください。
- `end_range`: 要素を抽出する終了範囲。BIGINT 値でなければなりません。`end_range` が `start_range` 以下の場合、NULL が返されます。例 3 を参照してください。

## 戻り値

BITMAP 型の値を返します。入力パラメータが無効な場合は NULL が返されます。

## 使用上の注意

サブセットには `start_range` は含まれますが、`end_range` は含まれません。例 5 を参照してください。

## 例

以下の例では、bitmap_subset_in_range() の入力は [bitmap_from_string](./bitmap_from_string.md) の出力です。例えば、`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` は `1, 3, 5, 7, 9` を返します。bitmap_subset_in_range() はこの BITMAP 値を入力として受け取ります。

例 1: 1 から 4 の範囲にある要素値を持つ BITMAP 値からサブセット要素を取得します。この範囲には 1 と 3 が含まれます。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

例 2: 要素値が 1 から 100 の範囲にある BITMAP 値からサブセット要素を取得します。終了値が BITMAP 値の最大長を超えるため、一致するすべての要素が返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

例 3: 終了範囲 `3` が開始範囲 `4` 以下であるため、NULL が返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 4, 3)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例 4: 開始範囲 10 が BITMAP 値 `1,3,5,7,9` の最大長 (5) を超えているため、NULL が返されます。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例 5: 返されるサブセットには開始値 `1` が含まれますが、終了値 `3` は含まれません。

```plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,4,5,6,7,9'), 1, 3)) value;
+-------+
| value |
+-------+
| 1     |
+-------+
```

## 参照

[bitmap_subset_limit](./bitmap_subset_limit.md)
