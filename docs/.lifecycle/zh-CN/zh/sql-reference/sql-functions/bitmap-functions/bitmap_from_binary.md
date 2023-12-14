---
displayed_sidebar: "中文"
---

# bitmap_from_binary

## 功能

将特定格式的 Varbinary 类型的字符串转换为 Bitmap。可用于将 Bitmap 数据导入到 StarRocks。

该函数从 3.0 版本开始提供支持。

## 语法

```Haskell
BITMAP bitmap_from_binary(VARBINARY str)
```

## 参数说明

`str`：支持的数据类型为 VARBINARY。

## 返回值说明

返回的数据类型为 BITMAP。

## 示例

示例一：此函数可与其他 Bitmap 函数配合使用。

   ```Plain
   mysql> select bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string("0,1,2,3"))));
   +---------------------------------------------------------------------------------------+
   | bitmap_to_string(bitmap_from_binary(bitmap_to_binary(bitmap_from_string('0,1,2,3')))) |
   +---------------------------------------------------------------------------------------+
   | 0,1,2,3                                                                               |
   +---------------------------------------------------------------------------------------+
   1 行返回（用时 0.01 秒）
   ```