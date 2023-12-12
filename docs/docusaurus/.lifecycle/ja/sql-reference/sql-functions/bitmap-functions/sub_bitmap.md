---
displayed_sidebar: "Japanese"
---

# sub_bitmap

## 説明

`offset` で指定された位置から始まる BITMAP 値 `src` から `len` 要素を取得します。出力される要素は `src` の部分集合です。

この関数は、ページネーション付きクエリなどのシナリオで主に使用されます。v2.5 からサポートされています。

この関数は、[bitmap_subset_limit](./bitmap_subset_limit.md) に類似しています。異なるのは、この関数がオフセットから要素を取得するのに対し、bitmap_subset_limit は要素値 (`start_range`) から要素を取得します。

## 構文

```Haskell
BITMAP sub_bitmap(BITMAP src, BIGINT offset, BIGINT len)
```

## パラメータ

- `src`: 要素を取得したい BITMAP 値です。
- `offset`: 開始位置です。BIGINT 値である必要があります。`offset` を使用する際は、次の点に注意してください:
  - オフセットは 0 から始まります。
  - 負のオフセットは右から左にカウントされます。例 3 と例 4を参照してください。
  - `offset` で指定された開始位置が実際の BITMAP 値の長さを超えると、NULL が返されます。例 6 を参照してください。
- `len`: 取得する要素の数です。1 以上の BIGINT 値である必要があります。一致する要素の数が `len` の値よりも少ない場合、すべての一致する要素が返されます。例 2、3、7 を参照してください。

## 戻り値

BITMAP 型の値を返します。入力パラメータのいずれかが無効な場合は NULL が返されます。

## 例

以下の例では、sub_bitmap() の入力は [bitmap_from_string](./bitmap_from_string.md) の出力です。例えば、`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` は `1, 3, 5, 7, 9` を返します。sub_bitmap() はこの BITMAP 値を入力とします。

Example 1: オフセットを 0 に設定した状態で BITMAP 値から二つの要素を取得します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

Example 2: オフセットを 0 に設定した状態で BITMAP 値から 100 個の要素を取得します。100 は BITMAP 値の長さを超えており、一致する要素はすべて返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

Example 3: オフセットを -3 に設定した状態で BITMAP 値から 100 個の要素を取得します。100 は BITMAP 値の長さを超えており、一致する要素はすべて返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+
```

Example 4: オフセットを -3 に設定した状態で BITMAP 値から二つの要素を取得します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+
```

Example 5: `-10` は `len` の無効な入力であるため、NULL が返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

Example 6: オフセット 5 で指定された開始位置が BITMAP 値 `1,3,5,7,9` の長さを超えています。NULL が返されます。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, 1)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

Example 7: `len` を 5 に設定しましたが、条件に一致する要素は二つだけです。この二つの要素がすべて返されます。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -2, 5)) value;
+-------+
| value |
+-------+
| 7,9   |
+-------+
```