---
displayed_sidebar: "Chinese"
---

# bitmap_empty

## 描述

返回一个空的位图。主要用于在插入或流加载过程中填充默认值。例如：

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,v1,v2=bitmap_empty()" \
    http://host:8410/api/test/testDb/_stream_load
```

## 语法

```Haskell
BITMAP BITMAP_EMPTY()
```

## 例子

```Plain Text
MySQL > select bitmap_count(bitmap_empty());
+------------------------------+
| bitmap_count(bitmap_empty()) |
+------------------------------+
|                            0 |
+------------------------------+
```

## 关键字

BITMAP_EMPTY, BITMAP