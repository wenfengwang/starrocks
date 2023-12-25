---
displayed_sidebar: English
---

# bitmap_to_string

## 説明

入力されたビットマップをコンマ (,) で区切られた文字列に変換します。この文字列にはビットマップの全ビットが含まれます。入力が null の場合は、null が返されます。

## 構文

```Haskell
VARCHAR BITMAP_TO_STRING(BITMAP input)
```

## パラメーター

`input`: 変換するビットマップです。

## 戻り値

VARCHAR 型の値を返します。

## 例

例 1: 入力が null で、null が返されます。

```Plain Text
MySQL > select bitmap_to_string(null);
+------------------------+
| bitmap_to_string(NULL) |
+------------------------+
| NULL                   |
+------------------------+
```

例 2: 入力が空で、空文字列が返されます。

```Plain Text
MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+
```

例 3: 1 ビットを含むビットマップを文字列に変換します。

```Plain Text
MySQL > select bitmap_to_string(to_bitmap(1));
+--------------------------------+
| bitmap_to_string(to_bitmap(1)) |
+--------------------------------+
| 1                              |
+--------------------------------+
```

例 4: 2 ビットを含むビットマップを文字列に変換します。

```Plain Text
MySQL > select bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2)));
+---------------------------------------------------------+
| bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2))) |
+---------------------------------------------------------+
| 1,2                                                     |
+---------------------------------------------------------+
```
