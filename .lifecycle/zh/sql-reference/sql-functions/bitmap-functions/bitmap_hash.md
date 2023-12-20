---
displayed_sidebar: English
---

# 位图哈希

## 描述

为任意类型的输入计算一个32位的哈希值，并返回包含该哈希值的位图。它主要用于流式加载任务，用于将非整数字段导入StarRocks表的位图字段。例如：

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
