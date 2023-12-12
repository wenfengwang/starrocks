```yaml
---
displayed_sidebar: "Japanese"
---

# array_to_bitmap

## 説明

配列をBITMAP値に変換します。この関数はv2.3からサポートされています。

## 構文

```Haskell
BITMAP array_to_bitmap(array)
```

## パラメータ

`array`: 配列内の要素はINT、TINYINT、またはSMALLINTタイプであることができます。

## 戻り値

BITMAPタイプの値を返します。

## 使用上の注意

- 入力配列の要素のデータ型がSTRINGやDECIMALなど無効な場合、エラーが返されます。

- 空の配列が入力された場合、空のBITMAP値が返されます。

- `NULL`が入力された場合、`NULL`が返されます。

## 例

Example 1: 配列をBITMAP値に変換します。BITMAP値は表示できないため、この関数は`bitmap_to_array`にネストする必要があります。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([1,2,3]));
+-------------------------------------------+
| bitmap_to_array(array_to_bitmap([1,2,3])) |
+-------------------------------------------+
| [1,2,3]                                   |
+-------------------------------------------+
```

Example 2: 空の配列が入力されたら、空の配列が返されます。

```Plain
MySQL > select bitmap_to_array(array_to_bitmap([]));
+--------------------------------------+
| bitmap_to_array(array_to_bitmap([])) |
+--------------------------------------+
| []                                   |
+--------------------------------------+
```

Example 3: `NULL`が入力されたら、`NULL`が返されます。

```Plain
MySQL > select array_to_bitmap(NULL);
+-----------------------+
| array_to_bitmap(NULL) |
+-----------------------+
| NULL                  |
+-----------------------+
```