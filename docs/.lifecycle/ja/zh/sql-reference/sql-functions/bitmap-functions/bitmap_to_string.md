---
displayed_sidebar: Chinese
---

# bitmap_to_string

## 機能

ビットマップをコンマ区切りの文字列に変換します。文字列にはビットマップのすべてのビット位置が含まれます。入力がnullの場合はnullを返します。

## 文法

```Haskell
BITMAP_TO_STRING(input)
```

## パラメータ説明

`input`: サポートされるデータ型は BITMAP です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
select bitmap_to_string(null);
+------------------------+
| bitmap_to_string(NULL) |
+------------------------+
| NULL                   |
+------------------------+

select bitmap_to_string(bitmap_empty());
+----------------------------------+
| bitmap_to_string(bitmap_empty()) |
+----------------------------------+
|                                  |
+----------------------------------+

select bitmap_to_string(to_bitmap(1));
+--------------------------------+
| bitmap_to_string(to_bitmap(1)) |
+--------------------------------+
| 1                              |
+--------------------------------+

select bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2)));
+---------------------------------------------------------+
| bitmap_to_string(bitmap_or(to_bitmap(1), to_bitmap(2))) |
+---------------------------------------------------------+
| 1,2                                                     |
+---------------------------------------------------------+

```
