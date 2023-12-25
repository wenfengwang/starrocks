---
displayed_sidebar: English
---

# bitmap_empty

## 説明

空のビットマップを返します。これは主に、挿入やストリームロード時にデフォルト値を設定するために使用されます。例えば：

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,v1,v2=bitmap_empty()" \
    http://host:8410/api/test/testDb/_stream_load
```

## 構文

```Haskell
BITMAP BITMAP_EMPTY()
```

## 例

```Plain Text
MySQL > select bitmap_count(bitmap_empty());
+------------------------------+
| bitmap_count(bitmap_empty()) |
+------------------------------+
|                            0 |
+------------------------------+
```

## キーワード

BITMAP_EMPTY、BITMAP
