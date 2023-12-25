---
displayed_sidebar: Chinese
---

# to_bitmap

## 機能

入力は0〜18446744073709551615の範囲のunsigned bigintで、出力はその要素を含むbitmapです。入力値がこの範囲外の場合は、NULLが返されます。

この関数は主にStream Loadのインポート時に整数型のフィールドをStarRocksテーブルのbitmapフィールドにインポートするために使用されます。以下の例を参照してください:

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=to_bitmap(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

## 文法

```Haskell
TO_BITMAP(expr)
```

## パラメータ説明

`expr`: サポートされるデータタイプはunsigned BIGINTです。

## 戻り値の説明

戻り値のデータタイプはBITMAPです。

## 例

```Plain Text
select bitmap_count(to_bitmap(10));
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
