---
displayed_sidebar: Chinese
---

# array_to_bitmap

## 機能

array 型を bitmap 型に変換します。この関数はバージョン 2.3 からサポートされています。

## 文法

```Haskell
array_to_bitmap(array)
```

## パラメータ説明

`array`: array の要素でサポートされるデータ型には INT、TINYINT、SMALLINT が含まれます。

## 戻り値の説明

BITMAP 型の値を返します。

## 注意事項

- 入力された array が不正なデータ型（例：STRING、DECIMAL など）の場合、エラーが返されます。

- 空の array を入力した場合、空の bitmap を返します。

- NULL を入力した場合、NULL を返します。

## 例

例1：array を入力して bitmap に変換します。bitmap 型は表示できないため、説明を容易にするために `bitmap_to_array` をネストしています。

```Plain Text
MySQL > select bitmap_to_array(array_to_bitmap([1,2,3]));
+-------------------------------------------+
| bitmap_to_array(array_to_bitmap([1,2,3])) |
+-------------------------------------------+
| [1,2,3]                                   |
+-------------------------------------------+
```

例2：空の array を入力します。

```Plain Text
MySQL > select bitmap_to_array(array_to_bitmap([]));
+--------------------------------------+
| bitmap_to_array(array_to_bitmap([])) |
+--------------------------------------+
| []                                   |
+--------------------------------------+
```

例3：NULL を入力します。

```Plain Text
MySQL > select array_to_bitmap(NULL);
+-----------------------+
| array_to_bitmap(NULL) |
+-----------------------+
| NULL                  |
+-----------------------+
```
