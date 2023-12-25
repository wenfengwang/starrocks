---
displayed_sidebar: English
---

# array_to_bitmap

## 説明

配列をBITMAP値に変換します。この関数はv2.3からサポートされています。

## 構文

```Haskell
BITMAP array_to_bitmap(array)
```

## パラメーター

`array`: 配列内の要素はINT、TINYINT、またはSMALLINT型である必要があります。

## 戻り値

BITMAP型の値を返します。

## 使用上の注意

- 入力配列内の要素のデータ型がSTRINGやDECIMALなど、無効な場合はエラーが返されます。

- 空の配列を入力すると、空のBITMAP値が返されます。

- `NULL`が入力された場合、`NULL`が返されます。

## 例

例 1: 配列をBITMAP値に変換します。この関数はBITMAP値を表示できないため、`bitmap_to_array`関数でネストする必要があります。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([1,2,3]));
+-------------------------------------------+
| bitmap_to_array(array_to_bitmap([1,2,3])) |
+-------------------------------------------+
| [1,2,3]                                   |
+-------------------------------------------+
```

例 2: 空の配列を入力すると、空の配列が返されます。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([]));
+--------------------------------------+
| bitmap_to_array(array_to_bitmap([])) |
+--------------------------------------+
| []                                   |
+--------------------------------------+
```

例 3: `NULL`を入力すると、`NULL`が返されます。

```Plain
MySQL > select array_to_bitmap(NULL);
+-----------------------+
| array_to_bitmap(NULL) |
+-----------------------+
| NULL                  |
+-----------------------+
```
