---
displayed_sidebar: Chinese
---

# bitmap_subset_limit

## 機能

指定された開始値に基づいて、BITMAP から指定された数の要素を抽出します。返される要素は Bitmap のサブセットです。

この関数はバージョン 3.1 からサポートされており、主にページングクエリのシナリオで使用されます。

この関数は [sub_bitmap](./sub_bitmap.md) と機能が似ていますが、sub_bitmap が指定するのは offset であり、bitmap_subset_limit は開始値を指定します。

## 文法

```Haskell
BITMAP bitmap_subset_limit(BITMAP src, BIGINT start_range, BIGINT limit)
```

## パラメータ説明

- `src`: 抽出対象の bitmap。
- `start_range`: 開始値を指定するために使用され、BIGINT 型でなければなりません。指定された開始値が Bitmap の最大長を超え、かつ `limit` が正の数である場合、NULL を返します。例四を参照してください。
- `limit`: `start_range` から始めて抽出する要素の数で、BIGINT 型でなければなりません。値が負の場合は、`start_range` から右に向かってカウントし、条件に合う要素の数が `limit` の値より少ない場合は、条件に合うすべての要素を返します。

## 戻り値の説明

入力された BITMAP のサブセットを返します。いずれかの入力パラメータが無効な場合は、NULL を返します。

## 使用説明

- 返されるサブセットには `start_range` が含まれます。
- `limit` が負の場合、`start_range` から右に向かってカウントし、`limit` の数の要素を返します。例三を参照してください。

## 例

以下の例では、bitmap_subset_in_range() 関数の入力値は [bitmap_from_string](./bitmap_from_string.md) 関数で計算された結果に基づいています。例えば、例中の `bitmap_from_string('1,1,3,1,5,3,5,7,7,9')` の実際の BITMAP 値は `1, 3, 5, 7, 9` です。bitmap_subset_in_range() はこの値に基づいて計算を行います。

例一：開始値 1 から始めて、Bitmap から 4 つの要素を抽出します。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+---------+
|  value  |
+---------+
| 1,3,5,7 |
+---------+
```

例二：開始値 1 から始めて、Bitmap から 100 個の要素を抽出します。100 は Bitmap の最大長を超えているため、すべての一致する要素を返します。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

例三：開始値 5 から始めて、右に向かってカウントし、Bitmap から 2 つの要素を抽出します。

```Plaintext
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 5, -2)) value;
+-----------+
| value     |
+-----------+
| 3,5       |
+-----------+
```

例四：開始値 10 が Bitmap の最大長を超え、`limit` が正の数であるため、NULL を返します。

```Plain
select bitmap_to_string(bitmap_subset_limit(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

## 関連参照

[sub_bitmap](./sub_bitmap.md)、 [bitmap_subset_in_range](./bitmap_subset_in_range.md)
