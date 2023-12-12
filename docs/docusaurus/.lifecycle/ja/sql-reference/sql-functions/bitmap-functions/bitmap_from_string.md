---
displayed_sidebar: "Japanese"
---

# bitmap_from_string（文字列からビットマップへ）

## 説明

文字列をBITMAPに変換します。文字列は、カンマで区切られた一連のUINT32番号から構成されています。例えば、文字列 "0, 1, 2" は、ビット 0、1、2 がセットされたビットマップに変換されます。入力フィールドが無効な場合、NULLが返されます。

この機能は、変換中に入力文字列の重複を削除します。他の関数と共に使用する必要があります。例えば、[bitmap_to_string](bitmap_to_string.md) などです。

## 構文

```Haskell
BITMAP BITMAP_FROM_STRING(VARCHAR input)
```

## 例

```Plain Text

-- 入力が空であり、空の値が返されます。

MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+

-- `0,1,2` が返されます。

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 2"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 2')) |
+-------------------------------------------------+
| 0,1,2                                           |
+-------------------------------------------------+

-- `-1` は無効な入力であり、NULLが返されます。

MySQL > select bitmap_to_string(bitmap_from_string("-1, 0, 1, 2"));
+-----------------------------------+
| bitmap_from_string('-1, 0, 1, 2') |
+-----------------------------------+
| NULL                              |
+-----------------------------------+

-- 入力文字列は重複が削除されます。

MySQL > select bitmap_to_string(bitmap_from_string("0, 1, 1"));
+-------------------------------------------------+
| bitmap_to_string(bitmap_from_string('0, 1, 1')) |
+-------------------------------------------------+
| 0,1                                             |
+-------------------------------------------------+
```

## キーワード

BITMAP_FROM_STRING, BITMAP