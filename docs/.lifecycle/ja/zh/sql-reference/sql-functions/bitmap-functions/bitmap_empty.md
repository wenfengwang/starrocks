---
displayed_sidebar: Chinese
---

# bitmap_empty

## 機能

空のbitmapを返し、主にinsertやstream load時にデフォルト値として使用されます。以下の例を参照してください:

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,v1,v2=bitmap_empty()" \
    http://host:8410/api/test/testDb/_stream_load
```

## 文法

```Haskell
BITMAP_EMPTY()
```

## パラメータ説明

なし

## 戻り値の説明

戻り値のデータ型はBITMAPです。

## 例

```Plain Text
MySQL > select bitmap_count(bitmap_empty());
+------------------------------+
| bitmap_count(bitmap_empty()) |
+------------------------------+
|                            0 |
+------------------------------+
```
