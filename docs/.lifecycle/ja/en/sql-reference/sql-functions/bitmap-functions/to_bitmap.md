---
displayed_sidebar: English
---

# to_bitmap

## 説明

入力は0から18446744073709551615までの範囲の符号なしbigintで、出力はこの要素を含むビットマップです。この関数は主に、整数フィールドをStarRocksテーブルのbitmapフィールドにインポートするためのストリームロードタスクで使用されます。例えば：

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=to_bitmap(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

## 構文

```Haskell
BITMAP TO_BITMAP(expr)
```

## 例

```Plain Text
MySQL > select bitmap_count(to_bitmap(10));
+-----------------------------+
| bitmap_count(to_bitmap(10)) |
+-----------------------------+
|                           1 |
+-----------------------------+

select bitmap_to_string(to_bitmap(10));
+---------------------------------+
| bitmap_to_string(to_bitmap(10)) |
+---------------------------------+
| 10                              |
+---------------------------------+

select bitmap_to_string(to_bitmap(-5));
+---------------------------------+
| bitmap_to_string(to_bitmap(-5)) |
+---------------------------------+
| NULL                            |
+---------------------------------+

select bitmap_to_string(to_bitmap(null));
+-----------------------------------+
| bitmap_to_string(to_bitmap(NULL)) |
+-----------------------------------+
| NULL                              |
+-----------------------------------+
```

## キーワード

TO_BITMAP, BITMAP
