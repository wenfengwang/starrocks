---
displayed_sidebar: "Chinese"
---

# array_slice

## 描述

返回数组的一个切片。该函数从`offset`指定的位置拦截`input`中的`length`个元素。

## 语法

```Haskell
array_slice(input, offset, length)
```

## 参数

- `input`：要拦截切片的数组。此函数支持以下类型的数组元素：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE 和 JSON。**从 2.5 版本开始支持 JSON。**

- `offset`：要拦截元素的位置。有效值从`1`开始。必须是 BIGINT 类型的值。

- `length`：要拦截的切片长度。必须是 BIGINT 类型的值。

## 返回值

返回与`input`参数指定的数组具有相同数据类型的数组。

## 使用注意事项

- 偏移量从 1 开始。
- 如果指定的长度超出了可拦截的实际元素数量，将返回所有匹配的元素。请参见示例 4。

## 示例

示例 1：从第三个元素开始，拦截 2 个元素。

```Plain
mysql> select array_slice([1,2,4,5,6], 3, 2) as res;
+-------+
| res   |
+-------+
| [4,5] |
+-------+
```

示例 2：从第一个元素开始，拦截 2 个元素。

```Plain
mysql> select array_slice(["sql","storage","query","execute"], 1, 2) as res;
+-------------------+
| res               |
+-------------------+
| ["sql","storage"] |
+-------------------+
```

示例 3：将空元素视为普通值。

```Plain
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 3) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```

示例 4：从第三个元素开始，拦截 5 个元素。

此函数意图拦截 5 个元素，但从第三个元素开始只有 3 个元素。因此，将返回所有这 3 个元素。

```Plain
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 5) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```