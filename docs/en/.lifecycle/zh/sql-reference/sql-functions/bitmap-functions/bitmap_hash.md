---
displayed_sidebar: English
---

# bitmap_hash

## 描述

计算任何类型输入的 32 位哈希值，并返回包含该哈希值的位图。它主要用于流加载任务，以将非整型字段导入StarRocks表的bitmap字段。例如：

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,device_id, device_id=bitmap_hash(device_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

## 语法

```Haskell
BITMAP BITMAP_HASH(expr)
```

## 示例

```Plain
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

## 关键字

BITMAP_HASH，BITMAP