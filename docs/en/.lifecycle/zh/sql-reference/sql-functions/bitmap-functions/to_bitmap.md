---
displayed_sidebar: English
---

# to_bitmap

## 描述

输入为无符号的 bigint，取值范围为 0 到 18446744073709551615，输出为包含该元素的位图。此函数主要用于流加载任务，将整数字段导入到 StarRocks 表的位图字段中。例如：

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=to_bitmap(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

## 语法

```Haskell
BITMAP TO_BITMAP(expr)
```

## 例子

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

## 关键词

TO_BITMAP，BITMAP