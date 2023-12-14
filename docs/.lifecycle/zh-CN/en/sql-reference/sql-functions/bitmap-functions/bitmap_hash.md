---
displayed_sidebar: "Chinese"
---

# bitmap_hash

## 描述

计算任何类型输入的32位哈希值，并返回包含哈希值的位图。主要用于流加载任务，以将非整数字段导入到StarRocks表的位图字段中。例如：

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

## 关键词

BITMAP_HASH,BITMAP