---
displayed_sidebar: "Japanese"
---

# sub_bitmap

## 説明

`offset` で指定された位置から始まる BITMAP 値 `src` から `len` 要素を取得します。出力される要素は `src` のサブセットです。

この関数は、ページネーションクエリなどのシナリオで主に使用されます。v2.5 からサポートされています。

この関数は [bitmap_subset_limit](./bitmap_subset_limit.md) と似ていますが、この関数はオフセットから要素を取得するのに対して、bitmap_subset_limit は要素値 (`start_range`) から要素を取得します。

## 構文

```Haskell
BITMAP sub_bitmap(BITMAP src, BIGINT offset, BIGINT len)
```

## パラメータ

- `src`: 要素を取得したい BITMAP 値です。
- `offset`: 開始位置です。BIGINT 値である必要があります。`offset` を使用する際には以下の点に注意してください:
  - オフセットは 0 から始まります。
  - 負のオフセットは右から左に数えられます。例 3 と 4 を参照してください。
  - `offset` で指定された開始位置が BITMAP 値の実際の長さを超える場合、NULL が返されます。例 6 を参照してください。
- `len`: 取得する要素の数です。1 以上の BIGINT 値である必要があります。一致する要素の数が `len` の値よりも少ない場合、すべての一致する要素が返されます。例 2、3、7 を参照してください。

## 戻り値

BITMAP 型の値が返されます。入力パラメータのいずれかが無効な場合は NULL が返されます。

## 例

以下の例では、sub_bitmap() の入力は [bitmap_from_string](./bitmap_from_string.md) の出力です。たとえば、`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` は `1, 3, 5, 7, 9` を返します。sub_bitmap() はこの BITMAP 値を入力として受け取ります。

例 1: オフセットを 0 に設定して BITMAP 値から 2 つの要素を取得します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

例 2: オフセットを 0 に設定して BITMAP 値から 100 の要素を取得します。100 は BITMAP 値の長さを超えており、すべての一致する要素が返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

例 3: オフセットを -3 に設定して BITMAP 値から 100 の要素を取得します。100 は BITMAP 値の長さを超えており、すべての一致する要素が返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+
```

例 4: オフセットを -3 に設定して BITMAP 値から 2 つの要素を取得します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+
```

例 5: `-10` は `len` の無効な入力なので NULL が返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例 6: オフセット 5 で指定された開始位置が BITMAP 値 `1,3,5,7,9` の長さを超えているため、NULL が返されます。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, 1)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例 7: `len` を 5 に設定しますが、条件に一致する要素は 2 つだけです。これらの 2 つの要素がすべて返されます。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -2, 5)) value;
+-------+
| value |
+-------+
| 7,9   |
+-------+
```
