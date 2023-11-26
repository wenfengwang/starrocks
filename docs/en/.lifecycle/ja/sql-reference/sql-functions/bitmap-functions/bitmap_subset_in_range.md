---
displayed_sidebar: "Japanese"
---

# bitmap_subset_in_range

## 説明

`start_range` と `end_range`（排他的）の範囲内の要素を Bitmap 値から抽出します。出力される要素は Bitmap 値のサブセットです。

この関数は、ページネーションクエリなどのシナリオで主に使用されます。v3.1 からサポートされています。

## 構文

```Haskell
BITMAP bitmap_subset_in_range(BITMAP src, BIGINT start_range, BIGINT end_range)
```

## パラメータ

- `src`: 要素を取得するための Bitmap 値。
- `start_range`: 要素を抽出する開始範囲。BIGINT 値である必要があります。指定した開始範囲が BITMAP 値の最大長を超える場合、NULL が返されます。例 4 を参照してください。
- `end_range`: 要素を抽出する終了範囲。BIGINT 値である必要があります。`end_range` が `start_range` と等しいかそれ以下の場合、NULL が返されます。例 3 を参照してください。

## 戻り値

BITMAP 型の値が返されます。入力パラメータのいずれかが無効な場合は、NULL が返されます。

## 使用上の注意

サブセットの要素には `start_range` が含まれ、`end_range` は含まれません。例 5 を参照してください。

## 例

以下の例では、bitmap_subset_in_range() の入力は [bitmap_from_string](./bitmap_from_string.md) の出力です。たとえば、`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` は `1, 3, 5, 7, 9` を返します。bitmap_subset_in_range() はこの BITMAP 値を入力として受け取ります。

例 1: 要素の値が 1 から 4 の範囲内の BITMAP 値からサブセットの要素を取得します。この範囲内の値は 1 と 3 です。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

例 2: 要素の値が 1 から 100 の範囲内の BITMAP 値からサブセットの要素を取得します。終了値が BITMAP 値の最大長を超えており、すべての一致する要素が返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

例 3: 終了範囲 `3` が開始範囲 `4` よりも小さいため、NULL が返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 4, 3)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例 4: 開始範囲 10 が BITMAP 値 `1,3,5,7,9` の最大長（5）を超えているため、NULL が返されます。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例 5: 返されるサブセットには開始値 `1` が含まれ、終了値 `3` は含まれません。

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
