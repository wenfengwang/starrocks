```yaml
---
displayed_sidebar: "Japanese"
---

# bitmap_subset_in_range

## 説明

`start_range` と `end_range`（排他）の範囲内で Bitmap 値から要素を取得してインターセプトします。出力される要素は Bitmap 値のサブセットです。

この関数は主にページネーションされたクエリなどのシナリオに使用されます。v3.1 からサポートされています。

## 構文

```Haskell
BITMAP bitmap_subset_in_range(BITMAP src, BIGINT start_range, BIGINT end_range)
```

## パラメーター

- `src`: 要素を取得する Bitmap 値です。
- `start_range`: 要素をインターセプトする開始範囲です。BIGINT 値でなければなりません。指定された開始範囲が BITMAP 値の最大長を超えると、NULL が返されます。 Example 4 を参照してください。
- `end_range`: 要素をインターセプトする終了範囲です。BIGINT 値でなければなりません。`end_range` が `start range` と等しいかそれより小さい場合、NULL が返されます。 Example 3 を参照してください。

## 返り値

BITMAP 型の値が返されます。入力パラメーターのいずれかが無効な場合、NULL が返されます。

## 使用上の注意

サブセットの要素には `start_range` が含まれ、`end_range` が含まれません。 Example 5 を参照してください。

## 例

以下の例では、bitmap_subset_in_range() の入力は [bitmap_from_string](./bitmap_from_string.md) の出力です。例えば、`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` は `1, 3, 5, 7, 9` を返します。bitmap_subset_in_range() はこの BITMAP 値を入力とします。

Example 1: 範囲が1から4のBITMAP値からサブセット要素を取得します。この範囲内の値は1と3です。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

Example 2: 範囲が1から100のBITMAP値からサブセット要素を取得します。終了値がBITMAP値の最大長を超えており、すべての一致する要素が返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

Example 3: 終了範囲 `3` が開始範囲 `4` より小さいため、NULL が返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 4, 3)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

Example 4: 開始範囲10がBITMAP値 `1,3,5,7,9` の最大長（5）を超えています。NULL が返されます。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

Example 5: 返されたサブセットには開始値 `1` が含まれますが、終了値 `3` は含まれません。

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
```