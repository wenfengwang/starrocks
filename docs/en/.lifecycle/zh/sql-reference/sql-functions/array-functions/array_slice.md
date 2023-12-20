---
displayed_sidebar: English
---

# array_slice

## 描述

返回数组的一个切片。此函数从 `input` 中 `offset` 指定的位置开始，截取 `length` 个元素。

## 语法

```Haskell
array_slice(input, offset, length)
```

## 参数

- `input`: 您想要截取切片的数组。此函数支持以下类型的数组元素：BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, FLOAT, DOUBLE, VARCHAR, DECIMALV2, DATETIME, DATE 和 JSON。**JSON 从 2.5 版本开始支持。**

- `offset`: 开始截取元素的位置。有效值从 `1` 开始。它必须是 BIGINT 类型的值。

- `length`: 您想要截取的切片长度。它必须是 BIGINT 类型的值。

## 返回值

返回一个数组，其数据类型与 `input` 参数指定的数组相同。

## 使用说明

- 偏移量从 1 开始。
- 如果指定的长度超过了实际能够截取的元素数量，将返回所有可匹配的元素。参见示例 4。

## 示例

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

示例 3：空元素被当作正常值处理。

```Plain
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 3) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```

示例 4：从第三个元素开始截取 5 个元素。

此函数意图截取 5 个元素，但从第三个元素开始仅有 3 个元素。因此，这 3 个元素都被返回。

```Plain
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 5) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```