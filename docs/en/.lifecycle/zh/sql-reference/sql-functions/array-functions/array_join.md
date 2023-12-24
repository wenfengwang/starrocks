---
displayed_sidebar: English
---

# array_join

## 描述

将数组的元素连接成一个字符串。

## 语法

```Haskell
array_join(array, sep[, null_replace_str])
```

## 参数

- `array`：要连接其元素的数组。仅支持 ARRAY 数据类型。

- `sep`：用于分隔串联数组元素的分隔符。仅支持 VARCHAR 数据类型。

- `null_replace_str`：用于替换 `NULL` 值的字符串。仅支持 VARCHAR 数据类型。

## 返回值

返回 VARCHAR 数据类型的值。

## 使用说明

- `array` 参数的值必须是一维数组。

- `array` 参数不支持 DECIMAL 值。

- 如果将 `sep` 参数设置为 `NULL`，则返回值为 `NULL`。

- 如果未指定 `null_replace_str` 参数，则 `NULL` 值将被丢弃。

- 如果将 `null_replace_str` 参数设置为 `NULL`，则返回值为 `NULL`。

## 例子

示例 1：连接数组的元素。在此示例中，数组中的 `NULL` 值被丢弃，串联的数组元素由下划线（`_`）分隔。

```plaintext
mysql> select array_join([1, 3, 5, null], '_');

+-------------------------------+

| array_join([1,3,5,NULL], '_') |

+-------------------------------+

| 1_3_5                         |

+-------------------------------+
```

示例 2：连接数组的元素。在此示例中，数组中的 `NULL` 值被替换为 `NULL` 字符串，串联的数组元素由下划线（`_`）分隔。

```plaintext
mysql> select array_join([1, 3, 5, null], '_', 'NULL');

+---------------------------------------+

| array_join([1,3,5,NULL], '_', 'NULL') |

+---------------------------------------+

| 1_3_5_NULL                            |

+---------------------------------------+
```