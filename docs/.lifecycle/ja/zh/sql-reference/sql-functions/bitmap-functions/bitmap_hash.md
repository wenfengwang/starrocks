---
displayed_sidebar: Chinese
---

# bitmap_hash

## 機能

任意の型の入力に対して32ビットのハッシュ値を計算し、そのハッシュ値を含むbitmapを返します。

主にstream loadのインポートで、整数型でないフィールドをStarRocksのテーブルのbitmapフィールドにインポートする際に使用されます。以下の例を参照してください:

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,device_id, device_id=bitmap_hash(device_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

## 文法

```Haskell
BITMAP_HASH(expr)
```

## パラメータ説明

`expr`: 任意のデータ型が指定できます。

## 戻り値の説明

戻り値のデータ型はBITMAPです。

## 例

```Plain Text
MySQL > select bitmap_count(bitmap_hash('hello'));
+------------------------------------+
| bitmap_count(bitmap_hash('hello')) |
+------------------------------------+
|                                  1 |
+------------------------------------+

select bitmap_to_string(bitmap_hash('hello'));
+----------------------------------------+
| bitmap_to_string(bitmap_hash('hello')) |
+----------------------------------------+
| 1321743225                             |
+----------------------------------------+
```
