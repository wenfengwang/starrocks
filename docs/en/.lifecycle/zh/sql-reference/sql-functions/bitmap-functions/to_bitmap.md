---
displayed_sidebar: English
---

# to_bitmap

## 描述

输入是无符号的 bigint，取值范围从0到18446744073709551615，输出是包含这个元素的 bitmap。这个函数主要用于流式加载任务，将整型字段导入StarRocks表的 bitmap 字段。例如：

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=to_bitmap(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

## 语法

```Haskell
BITMAP TO_BITMAP(expr)
```

## 示例

```Plain
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

## 关键字

TO_BITMAP, BITMAP