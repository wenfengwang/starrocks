```yaml
---
displayed_sidebar: "Japanese"
---

# ビットマップ_レンジ内のサブセット

## 説明

`start_range` と `end_range`（排他的）の範囲内の Bitmap 値から要素を取得します。出力要素は Bitmap 値のサブセットです。

この機能は主にページネーションされたクエリなどのシナリオで使用されます。v3.1 からサポートされています。

## 構文

```Haskell
BITMAP bitmap_subset_in_range(BITMAP src, BIGINT start_range, BIGINT end_range)
```

## パラメータ

- `src`: 要素を取得する Bitmap 値。
- `start_range`: 要素を取得する開始範囲。BIGINT 値でなければなりません。指定された開始範囲が BITMAP 値の最大長を超えると、NULL が返されます。Example 4 を参照してください。
- `end_range`: 要素を取得する終了範囲。BIGINT 値でなければなりません。`end_range` が `start range` 以下であるか、等しい場合は、NULL が返されます。Example 3 を参照してください。

## 戻り値

BITMAP 型の値を返します。入力パラメータのいずれかが無効な場合は、NULL が返されます。

## 使用上の注意

サブセット要素には `start_range` が含まれますが、`end_range` は含まれません。Example 5 を参照してください。

## 例

次の例では、bitmap_subset_in_range() の入力は [bitmap_from_string](./bitmap_from_string.md) の出力です。たとえば、`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` は `1, 3, 5, 7, 9` を返します。bitmap_subset_in_range() はこの BITMAP 値を入力とします。

Example 1: 要素の値が 1 から 4 の範囲内の BITMAP 値からサブセット要素を取得します。この範囲内の値は 1 と 3 です。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

Example 2: 要素の値が範囲 1 から 100 の BITMAP 値からサブセット要素を取得します。終了値が BITMAP 値の最大長を超えており、すべての一致する要素が返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

Example 3: 終了範囲 `3` が開始範囲 `4` よりも小さいため、NULL が返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 4, 3)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

Example 4: 開始範囲 10 が BITMAP 値 `1,3,5,7,9` の最大長（5）を超えています。NULL が返されます。

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