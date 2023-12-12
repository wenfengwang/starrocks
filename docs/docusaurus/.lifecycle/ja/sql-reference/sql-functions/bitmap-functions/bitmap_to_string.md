---
displayed_sidebar: "英語"
---

# bitmap_to_string

## 説明

入力ビットマップをカンマ（,）で区切られた文字列に変換します。この文字列にはビットマップの全ビットが含まれています。入力がnullの場合、nullを返します。

## 構文

```Haskell
VARCHAR BITMAP_TO_STRING(BITMAP input)
```

## パラメータ

`input`: 変換するビットマップです。

## 戻り値

VARCHAR型の値を返します。

## 例

例1: 入力がnullで、nullが返される場合です。

```Plain Text
MySQL > select bitmap_to_string(null);
+------------------------+
| bitmap_to_string(NULL) |
+------------------------+
| NULL                   |
+------------------------+
```

例2: 入力が空で、空の文字列が返される場合です。

```Plain Text
MySQL > select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+
```

例3: 1ビットを含むビットマップを文字列に変換する場合です。

```Plain Text
MySQL > select bitmap_to_string(to_bitmap(1));
+--------------------------------+
| bitmap_to_string(to_bitmap(1)) |
+--------------------------------+
| 1                              |
+--------------------------------+
```

例4: 2ビットを含むビットマップを文字列に変換する場合です。

```Plain Text
MySQL > select bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2)));
+---------------------------------------------------------+
| bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2))) |
+---------------------------------------------------------+
| 1,2                                                     |
+---------------------------------------------------------+
```