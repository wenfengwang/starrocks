---
displayed_sidebar: English
---

# array_slice

## 描述

返回数组的切片。此函数从`input`指定的位置截取`length`个元素。

## 语法

```Haskell
array_slice(input, offset, length)
```

## 参数

- `input`：要截取其切片的数组。此函数支持以下类型的数组元素：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE 和 JSON。 **从 2.5 开始支持 JSON。**

- `offset`：截取元素的起始位置。有效值从`1`开始。它必须是 BIGINT 值。

- `length`：要截取的切片的长度。它必须是 BIGINT 值。

## 返回值

返回一个与参数`input`指定的数组具有相同数据类型的数组。

## 使用说明

- 偏移量从 1 开始。
- 如果指定的长度超过可截取的实际元素数，则返回所有匹配的元素。请参阅示例 4。

## 例子

示例 1：从第三个元素开始截取 2 个元素。

```Plain
mysql> select array_slice([1,2,4,5,6], 3, 2) as res;
+-------+
| res   |
+-------+
| [4,5] |
+-------+
```

示例 2：从第一个元素开始截取 2 个元素。

```Plain
mysql> select array_slice(["sql","storage","query","execute"], 1, 2) as res;
+-------------------+
| res               |
+-------------------+
| ["sql","storage"] |
+-------------------+
```

示例 3：将 Null 元素视为普通值。

```Plain
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 3) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```

示例 4：从第三个元素开始截取 5 个元素。

此函数意图截取 5 个元素，但从第三个元素开始只有 3 个元素。因此，返回这 3 个元素。

```Plain
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 5) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```
