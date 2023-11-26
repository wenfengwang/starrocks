---
displayed_sidebar: "Japanese"
---

# bitmap_subset_limit

## 説明

指定された要素数のBITMAP値を、`start_range`から始まる要素値で切り取ります。出力される要素は`src`のサブセットです。

この関数は、ページネーションクエリなどのシナリオで主に使用されます。v3.1からサポートされています。

この関数は[sub_bitmap](./sub_bitmap.md)と似ていますが、この関数は要素値(`start_range`)から要素を切り取りますが、sub_bitmapはオフセットから要素を切り取ります。

## 構文

```Haskell
BITMAP bitmap_subset_limit(BITMAP src, BIGINT start_range, BIGINT limit)
```

## パラメータ

- `src`: 要素を取得するためのBITMAP値です。
- `start_range`: 要素を切り取る開始範囲です。BIGINT値である必要があります。指定された開始範囲がBITMAP値の最大要素を超え、かつ`limit`が正の場合、NULLが返されます。例4を参照してください。
- `limit`: `start_range`から開始して取得する要素の数です。負の制限は右から左に数えられます。一致する要素の数が`limit`の値よりも少ない場合、すべての一致する要素が返されます。

## 戻り値

BITMAP型の値が返されます。入力パラメータのいずれかが無効な場合は、NULLが返されます。

## 使用上の注意

- サブセットの要素には`start range`が含まれます。
- 負の制限は右から左に数えられます。例3を参照してください。

## 例

以下の例では、bitmap_subset_limit()の入力は[bitmap_from_string](./bitmap_from_string.md)の出力です。たとえば、`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')`は`1, 3, 5, 7, 9`を返します。bitmap_subset_limit()は、このBITMAP値を入力として受け取ります。

例1: 要素値が1から始まるBITMAP値から4つの要素を取得します。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+---------+
|  value  |
+---------+
| 1,3,5,7 |
+---------+
```

例2: 要素値が1から始まるBITMAP値から100個の要素を取得します。制限はBITMAP値の長さを超えており、すべての一致する要素が返されます。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

例3: 要素値が5から始まるBITMAP値から-2つの要素を取得します（右から左に数えます）。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, -2)) value;
+-----------+
| value     |
+-----------+
| 3,5       |
+-----------+
```

例4: 開始範囲10はBITMAP値`1,3,5,7,9`の最大要素を超えており、制限が正です。NULLが返されます。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```
