---
displayed_sidebar: "Chinese"
---

# to_bitmap

## 描述

输入为取值范围从0到18446744073709551615的无符号大整数，输出是包含此元素的位图。此函数主要用于流加载任务，将整数字段导入StarRocks表的位图字段中。例如：

```bash
cat 数据 | curl --location-trusted -u 用户:密码 -T - \
    -H "columns: dt,page,user_id, user_id=to_bitmap(user_id)" \
    http://主机:8410/api/test/testDb/_stream_load
```

## 语法

```Haskell
BITMAP TO_BITMAP(expr)
```

## 示例

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

TO_BITMAP,BITMAP