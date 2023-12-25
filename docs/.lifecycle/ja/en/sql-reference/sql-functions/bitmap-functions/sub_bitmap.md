---
displayed_sidebar: English
---

# sub_bitmap

## 説明

`offset`で指定された位置から始まる`len`要素をBITMAP値`src`から取り出します。出力要素は`src`のサブセットです。

この関数は、ページネーションされたクエリなどのシナリオで主に使用されます。v2.5からサポートされています。

この関数は[bitmap_subset_limit](./bitmap_subset_limit.md)に似ています。違いは、この関数がオフセットから要素を取り出すのに対し、bitmap_subset_limitは要素値（`start_range`）から要素を取り出す点です。

## 構文

```Haskell
BITMAP sub_bitmap(BITMAP src, BIGINT offset, BIGINT len)
```

## パラメーター

- `src`: 要素を取得したいBITMAP値です。
- `offset`: 開始位置です。BIGINT型の値でなければなりません。`offset`を使用する際は以下の点に注意してください：
  - オフセットは0から始まります。
  - 負のオフセットは右から左へと数えられます。例3と4を参照してください。
  - `offset`で指定された開始位置がBITMAP値の実際の長さを超える場合、NULLが返されます。例6を参照してください。
- `len`: 取得する要素の数です。1以上のBIGINT型の値でなければなりません。一致する要素の数が`len`の値より少ない場合、一致する全ての要素が返されます。例2、3、7を参照してください。

## 戻り値

BITMAP型の値を返します。入力パラメータが無効な場合はNULLが返されます。

## 例

以下の例では、sub_bitmap()の入力は[bitmap_from_string](./bitmap_from_string.md)の出力です。例えば、`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')`は`1, 3, 5, 7, 9`を返します。sub_bitmap()はこのBITMAP値を入力として受け取ります。

例1: オフセットを0に設定して、BITMAP値から2つの要素を取得します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

例2: オフセットを0に設定して、BITMAP値から100個の要素を取得します。100はBITMAP値の長さを超え、一致する全ての要素が返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

例3: オフセットを-3に設定して、BITMAP値から100個の要素を取得します。100はBITMAP値の長さを超え、一致する全ての要素が返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+
```

例4: オフセットを-3に設定して、BITMAP値から2つの要素を取得します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+
```

例5: `-10`は`len`の無効な入力なので、NULLが返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例6: オフセット5で指定された開始位置がBITMAP値`1,3,5,7,9`の長さを超えているため、NULLが返されます。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, 1)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例7: `len`は5に設定されていますが、条件に一致する要素は2つだけです。これら2つの要素が全て返されます。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -2, 5)) value;
+-------+
| value |
+-------+
| 7,9   |
+-------+
```
