---
displayed_sidebar: Chinese
---

# sub_bitmap

## 機能

`offset` で指定された開始位置から、BITMAP 型の `src` から `len` 個の要素を切り取り、その結果として `src` のサブセットを返します。この関数は主にページングクエリのシナリオで使用されます。この関数はバージョン 2.5 からサポートされています。

この関数は [bitmap_subset_limit](./bitmap_subset_limit.md) と機能が似ていますが、bitmap_subset_limit が開始値を指定するのに対し、sub_bitmap は offset を指定します。

## 文法

```Haskell
BITMAP sub_bitmap(BITMAP src, BIGINT offset, BIGINT len)
```

## パラメータ説明

- `src`: 切り取り対象の bitmap。
- `offset`: 開始位置を指定するために使用され、BIGINT 型をサポートします。Offset の使用にあたっての注意点：

  - Offset は 0 から始まります。
  - 負のオフセットは文字列の終わりから右から左へのカウントを意味します。例 3 と 4 を参照してください。
  - `offset` で指定された開始位置が BITMAP の実際の長さを超える場合は NULL を返します。例 6 を参照してください。

- `len`: 切り取る要素の数で、正の整数でなければなりません。そうでない場合は NULL を返します。条件に合う要素の数が `len` の値より少ない場合は、条件に合うすべての要素を返します。例 2、3、7 を参照してください。

## 戻り値の説明

戻り値の型は BITMAP です。入力パラメータが無効な場合は NULL を返します。

## 例

以下の例では、sub_bitmap() 関数の入力値は [bitmap_from_string](./bitmap_from_string.md) 関数によって計算された結果です。例えば、例中の `bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` は実際には BITMAP 値 `1, 3, 5, 7, 9` を出力します。sub_bitmap() はこの値を基に計算を行います。

例 1：オフセット 0 から始めて、BITMAP 値の中から 2 つの要素を切り取ります。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

例 2：オフセット 0 から始めて、BITMAP 値の中から 100 個の要素を切り取ります。BITMAP に 100 個の要素がないため、条件に合うすべての要素を出力します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

例 3：オフセット -3 から始めて、BITMAP 値の中から 100 個の要素を切り取ります。BITMAP に 100 個の要素がないため、条件に合うすべての要素を出力します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+
```

例 4：オフセット -3 から始めて、BITMAP 値の中から 2 つの要素を切り取ります。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+
```

例 5：`-10` は `len` に対する無効な入力で、NULL を返します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例 6：`offset` で指定された開始位置 5 が BITMAP 値 `1,3,5,7,9` の長さを超えているため、NULL を返します。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, 1)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例 7：実際に条件に合う要素が 2 つしかなく、`len` の値 5 より少ないため、条件に合うすべての要素を返します。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -2, 5)) value;
+-------+
| value |
+-------+
| 7,9   |
+-------+
```
