---
displayed_sidebar: "Japanese"
---

# bitmap_subset_limit

## 説明

指定された`start_range`から始まる要素を持つBITMAP値から指定された要素数を取り出します。 出力される要素は`src`のサブセットです。

この関数はページネーションクエリなどのシナリオで主に使用されます。 v3.1 からサポートされています。

この関数は[sub_bitmap](./sub_bitmap.md)に類似しています。違いは、この関数は要素値 (`start_range`) から要素を取るのに対し、sub_bitmapはオフセットから要素を取ります。

## 構文

```Haskell
BITMAP bitmap_subset_limit(BITMAP src, BIGINT start_range, BIGINT limit)
```

## パラメータ

- `src`: 要素を取得するBITMAP値です。
- `start_range`: 要素を取得する開始範囲です。BIGINT値でなければなりません。指定された開始範囲がBITMAP値の最大要素を超え、`limit`が正の場合、NULLが返されます。 Example 4を参照してください。
- `limit`: `start_range`から取得する要素数です。負の制限は右から左にカウントされます。一致する要素の数が`limit`の値より少ない場合、一致する要素すべてが返されます。

## 戻り値

BITMAPタイプの値が返されます。入力パラメータのいずれかが無効な場合、NULLが返されます。

## 使用上の注意

- サブセット要素には `start_range` が含まれます。
- 負の制限は右から左にカウントされます。例3を参照してください。

## 例

以下の例では、bitmap_subset_limit()の入力は[bitmap_from_string](./bitmap_from_string.md)の出力です。 例えば、 `bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` は `1, 3, 5, 7, 9` を返します。bitmap_subset_limit()はこのBITMAP値を入力とします。

例1: 要素値が1から始まるBITMAP値から4つの要素を取得します。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+---------+
|  value  |
+---------+
| 1,3,5,7 |
+---------+
```

例2: 要素値が1から始まるBITMAP値から100の要素を取得します。 制限はBITMAP値の長さを超え、一致する要素はすべて返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

例3: 要素値が5から始まるBITMAP値から-2つの要素を取得します（右から左にカウント）。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, -2)) value;
+-----------+
| value     |
+-----------+
| 3,5       |
+-----------+
```

例4: 開始範囲10はBITMAP値 `1,3,5,7,9`の最大要素を超えており、制限は正です。 NULLが返されます。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```