---
displayed_sidebar: "Japanese"
---

# bitmap_hash

## 説明

任意の入力の32ビットハッシュ値を計算し、ハッシュ値を含むビットマップを返します。これは主にストリームロードタスクで使用され、StarRocksテーブルの非整数フィールドをビットマップフィールドにインポートするために使用されます。例えば：

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,device_id, device_id=bitmap_hash(device_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

## 構文

```Haskell
BITMAP BITMAP_HASH(expr)
```

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

## キーワード

BITMAP_HASH,BITMAP
