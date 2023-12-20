---
displayed_sidebar: English
---

# 数组拼接

## 描述

将数组中的元素拼接成一个字符串。

## 语法

```Haskell
array_join(array, sep[, null_replace_str])
```

## 参数

- array：需要拼接其元素的数组。只支持 ARRAY 数据类型。

- sep：用来分隔拼接数组元素的分隔符。只支持 VARCHAR 数据类型。

- null_replace_str：用以替代 NULL 值的字符串。只支持 VARCHAR 数据类型。

## 返回值

返回一个 VARCHAR 数据类型的值。

## 使用须知

- array 参数必须是一维数组。

- array 参数不支持 DECIMAL 类型的值。

- 如果 sep 参数被设定为 NULL，则返回值为 NULL。

- 如果未指定 null_replace_str 参数，NULL 值会被忽略。

- 如果 null_replace_str 参数被设定为 NULL，则返回值为 NULL。

## 示例

示例 1：拼接数组中的元素。在这个例子中，数组里的 NULL 值会被忽略，拼接后的数组元素之间用下划线（_）分隔。

```plaintext
mysql> select array_join([1, 3, 5, null], '_');

+-------------------------------+

| array_join([1,3,5,NULL], '_') |

+-------------------------------+

| 1_3_5                         |

+-------------------------------+
```

示例 2：拼接数组中的元素。在这个例子中，数组里的 NULL 值会被替换成 NULL 字符串，拼接后的数组元素之间用下划线（_）分隔。

```plaintext
mysql> select array_join([1, 3, 5, null], '_', 'NULL');

+---------------------------------------+

| array_join([1,3,5,NULL], '_', 'NULL') |

+---------------------------------------+

| 1_3_5_NULL                            |

+---------------------------------------+
```
