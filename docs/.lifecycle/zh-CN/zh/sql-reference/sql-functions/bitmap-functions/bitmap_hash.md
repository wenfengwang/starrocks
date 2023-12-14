---
displayed_sidebar: "Chinese"
---

# bitmap_hash

## 功能

计算任意类型的输入的32位哈希值，并返回包含该哈希值的位图。

通常用于流式加载中，将非整数字段导入到StarRocks表中的位图字段，例如下面的示例：

```bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,device_id, device_id=bitmap_hash(device_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

## 语法

```Haskell
BITMAP_HASH(expr)
```

## 参数说明

`expr`: 可以是任意数据类型。

## 返回值说明

返回值的数据类型为BITMAP。

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