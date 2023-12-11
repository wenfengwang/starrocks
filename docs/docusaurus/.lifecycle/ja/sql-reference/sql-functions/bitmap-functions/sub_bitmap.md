```yaml
---
displayed_sidebar: "Japanese"
---

# sub_bitmap

## 説明

BITMAP値`src`から`offset`で指定された位置から始まる`len`要素を取得します。出力される要素は、`src`のサブセットです。

この機能は、主にページネーションクエリなどのシナリオで使用されます。v2.5からサポートされています。

この機能は[bm_subset_limit](./bitmap_subset_limit.md)に類似していますが、この機能はオフセットから要素を取得するのに対し、bitmap_subset_limitは要素値（`start_range`）から要素を取得します。

## 構文

```Haskell
BITMAP sub_bitmap(BITMAP src, BIGINT offset, BIGINT len)
```

## パラメータ

- `src`: 要素を取得したいBITMAP値です。
- `offset`: 開始位置です。BIGINT値である必要があります。`offset`を使用する場合は以下の点に注意してください：
  - オフセットは0から開始します。
  - 負のオフセットは右から左にカウントされます。Example 3および4を参照してください。
  - `offset`で指定された開始位置がBITMAP値の実際の長さを超える場合、NULLが返されます。Example 6を参照してください。
- `len`: 取得する要素の数です。1以上のBIGINT値である必要があります。一致する要素の数が`len`の値より少ない場合、すべての一致する要素が返されます。Example 2、3、および7を参照してください。

## 返り値

BITMAP型の値が返されます。入力パラメータのいずれかが無効な場合、NULLが返されます。

## 例

以下の例では、sub_bitmap()の入力は[bitmap_from_string](./bitmap_from_string.md)の出力です。例えば、`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')`は`1,3,5,7,9`を返します。sub_bitmap()はこのBITMAP値を入力とします。

Example 1: オフセットが0に設定されたBITMAP値から2つの要素を取得します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 2)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

Example 2: オフセットが0に設定されたBITMAP値から100の要素を取得します。100はBITMAP値の長さを超えており、一致する要素がすべて返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

Example 3: オフセットが-3に設定されたBITMAP値から100の要素を取得します。100はBITMAP値の長さを超えており、一致する要素がすべて返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 100)) value;
+-------+
| value |
+-------+
| 5,7,9 |
+-------+
```

Example 4: オフセットが-3に設定されたBITMAP値から2つの要素を取得します。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -3, 2)) value;
+-------+
| value |
+-------+
| 5,7   |
+-------+
```

Example 5: `-10`は`len`の無効な入力なので、NULLが返されます。

```Plaintext
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, -10)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

Example 6: オフセット5で指定された開始位置がBITMAP値`1,3,5,7,9`の長さを超えているため、NULLが返されます。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, 1)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

Example 7: `len`が5に設定されていますが、条件に一致する要素は2つだけです。これらの2つの要素がすべて返されます。

```Plain
select bitmap_to_string(sub_bitmap(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), -2, 5)) value;
+-------+
| value |
+-------+
| 7,9   |
+-------+
```