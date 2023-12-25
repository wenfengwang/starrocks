---
displayed_sidebar: Chinese
---

# bitmap_to_array

## 機能

BITMAP 中のすべての値を BIGINT 型の配列に組み合わせます。

## 文法

```Haskell
 `ARRAY<BIGINT>` BITMAP_TO_ARRAY (bitmap)
```

## パラメータ説明

`bitmap`: サポートされるデータ型は BITMAP です。

## 戻り値の説明

戻り値のデータ型は BIGINT 型の配列です。

## 例

```Plain text
select bitmap_to_array(bitmap_from_string("1, 7"));
+----------------------------------------------+
| bitmap_to_array(bitmap_from_string('1, 7'))  |
+----------------------------------------------+
| [1,7]                                        |
+----------------------------------------------+

select bitmap_to_array(NULL);
+-----------------------+
| bitmap_to_array(NULL) |
+-----------------------+
| NULL                  |
+-----------------------+
```
