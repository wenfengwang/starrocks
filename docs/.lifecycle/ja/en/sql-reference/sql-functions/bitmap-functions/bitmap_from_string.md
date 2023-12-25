---
displayed_sidebar: English
---

# bitmap_from_string

## 説明

文字列をBITMAPに変換します。文字列は、コンマで区切られたUINT32の数値のセットで構成されています。例えば、"0, 1, 2"という文字列は、ビット0、1、2がセットされたBitmapに変換されます。入力フィールドが無効な場合、NULLが返されます。

この関数は変換中に入力文字列の重複を排除します。[bitmap_to_string](bitmap_to_string.md)などの他の関数と組み合わせて使用する必要があります。

## 構文

```Haskell
BITMAP BITMAP_FROM_STRING(VARCHAR input)
```

## 例

```Plain Text

-- 入力が空で、空の値が返されます。

MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+

-- `0,1,2`が返されます。

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 2"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 2')) |
+-------------------------------------------------+
| 0,1,2                                           |
+-------------------------------------------------+

-- `-1`は無効な入力で、NULLが返されます。

MySQL > select bitmap_to_string(bitmap_from_string("-1, 0, 1, 2"));
+-----------------------------------+
| bitmap_from_string('-1, 0, 1, 2') |
+-----------------------------------+
| NULL                              |
+-----------------------------------+

-- 入力文字列は重複が排除されます。

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 1"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 1')) |
+-------------------------------------------------+
| 0,1                                             |
+-------------------------------------------------+
```

## キーワード

BITMAP_FROM_STRING, BITMAP
