---
displayed_sidebar: Chinese
---

# bitmap_subset_in_range

## 機能

Bitmapから指定範囲内の要素を返します。返される要素はBitmapのサブセットです。

この関数はバージョン3.1からサポートされており、主にページングクエリのシナリオで使用されます。

## 文法

```Haskell
BITMAP bitmap_subset_in_range(BITMAP src, BIGINT start_range, BIGINT end_range)
```

## パラメータ説明

- `src`: 対象となるbitmapを切り取ります。
- `start_range`: 範囲の開始値を指定するために使用され、BIGINT型でなければなりません。指定された開始値がBitmapの最大長を超える場合は、NULLを返します。例4を参照してください。`start_range`がBitmapに存在する場合、返される値には`start_range`が含まれます。
- `end_range`: 範囲の終了値を指定するために使用され、BIGINT型でなければなりません。`end_range`が`start_range`以下の場合は、NULLを返します。例3を参照してください。返される値には`end_range`は含まれません。

## 戻り値の説明

入力されたBITMAPのサブセットを返します。どの入力パラメータも無効な場合は、NULLを返します。

## 使用説明

返されるサブセットには`start_range`は含まれますが、`end_range`は含まれません。例5を参照してください。

## 例

以下の例では、bitmap_subset_in_range()関数の入力値は[bitmap_from_string](./bitmap_from_string.md)関数によって計算された結果です。例えば、例中の`bitmap_from_string('1,1,3,1,5,3,5,7,7,9')`の実際のBITMAP値は`1, 3, 5, 7, 9`です。bitmap_subset_in_range()はこの値を基に計算を行います。

例1：BITMAPから[1,4)の範囲内の要素を返します。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 1, 4)) value;
+-------+
| value |
+-------+
| 1,3   |
+-------+
```

例2：Bitmapから[1,100)の範囲内の要素を返します。100はBitmapの最大長を超えているため、すべての一致する要素を返します。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 0, 100)) value;
+-----------+
| value     |
+-----------+
| 1,3,5,7,9 |
+-----------+
```

例3：終了値3が開始値4より小さいため、NULLを返します。

```Plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 4, 3)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例4：開始値10がBitmapの最大長を超えているため、NULLを返します。

```Plain
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,3,5,7,7,9'), 10, 15)) value;
+-------+
| value |
+-------+
| NULL  |
+-------+
```

例5：Bitmapから[1,3)の範囲内の要素を返します。返されるサブセットには開始値1が含まれますが、終了値3は含まれません。

```plaintext
select bitmap_to_string(bitmap_subset_in_range(bitmap_from_string('1,1,3,1,5,4,5,6,7,9'), 1, 3)) value;
+-------+
| value |
+-------+
| 1     |
+-------+
```

## 関連文書

[bitmap_subset_limit](./bitmap_subset_limit.md)
